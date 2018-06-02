dagger2是用于进行依赖注入的框架。dagger1由square开发，而现在dagger2由google继续开发和维护。在google出的几个Android架构示例(https://github.com/googlesamples/android-architecture/tree/todo-mvp-dagger, https://github.com/googlesamples/android-architecture-components/tree/master/GithubBrowserSample)中也用到了dagger2，给人一种谷歌钦定的感觉。

那么，我们为什么要用到dagger呢？它是用来干嘛的？
本文将通过介绍dagger2的相关的概念和一些常用用法来初探dagger2并能够上手使用，一些深入的用法也会在后续的文章中介绍。
1.控制反转和依赖注入
首先，我们就要先介绍控制反转和依赖注入的概念了。
如果A类依赖B类的对象通常我们会怎么做呢？
public class A{
    private B b;
    public A(){
        b = new B();
    }
}
这是最通常的做法：在类A里new 一个B的对象作为成员变量，也就是A控制了B实例的生成，控制依赖。这样做的不足在于A需要了解B的细节，如果要替换B模块还需要相应的修改A。更严重的是，如果B的实例化依赖了其他类C/D/F...，则A类还需要管理C/D/F类的实例化过程，这样模块间的耦合度会很高，修改和替换难度都很大。
而控制反转(IoC)是一种设计思想，即设计好的对象的依赖由容器来控制，而非之前用对象来直接控制对象依赖。程序架构发生了“主从换位”的变化，应用程序被动的等待IoC容器来创建并注入它所需要的资源。
依赖注入(Dependency Injection)是控制反转的一种具体实现。组件之间依赖关系由注入器（容器）在运行期决定，即由容器动态的将某个依赖关系注入到组件之中。依赖注入将依赖统一管理，使得组件间耦合度降低，更容易拓展和修改，也能更好的控制组件的生命周期。如果某组件需要修改其依赖，如果新增加的依赖存在与依赖图中，那么只需修改该类就可以，注入器可以将新的依赖注入。
2.java依赖注入标准：JSR-330
java有一套依赖注入规范——JSR-330。该规范里定义了一些依赖注入相关的注解（javax.inject包下），dagger就是基于了这个规范，也就是dagger对外使用的annotation许多都是JSR-330下的。而其他的注入框架如Spring也支持JSR-330。上文的例子中，如果用支持JSR-330的依赖注入框架的话，最后实现的代码可以类似于：
public class A{
    @inject
    private B b;
}
在dagger2 中用的JSR-330标准注释有：@Inject @Qualifier @Scope @Named等。
值得注意的是，JSR-330并没有规范注入器，也就是说用不同的注入框架时，上述代码可以做到基本一致，都是符合JSR-330规范的，而不同框架注入器实现方式不同。
3.dagger2的基本结构



Dagger Injections Overview

(图摘自https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2)
使用dagger2进行依赖注入时，整个依赖关系如图。在dagger2中Component便是注入器，它维护了依赖图并向其他对象注入依赖。依赖图中的对象由Module提供，或是通过构造方法注入来获得，一些具体细节将在稍后介绍。

4.用Dagger2向类注入依赖
通过Dagger2将依赖注入到某个类中的时候，是用@Inject注解来实现的。@Inject注解为JSR-330标准中的一部分。
注入方式有两种：即@Inject可以标记域也可以标记构造方法。
（1）域注入
public class MainActivity extends Activity {
   @Inject 
   SharedPreferences mSharedPreferences;

   public void onCreate(Bundle savedInstance) {
       InjectorClass.inject(this);
   } 
注意，这里需要在适当的时机，如在组件的onCreate()中调用Component的inject()方法，此时将依赖注入到该对象中。此外，被@Inject标记的域不可为private，因为它需要有注入器Component进行赋值。
（2）构造方法注入
public class ProviderHandler {

    private ContentResolver mContentResolver;

    @Inject
    ProviderHandler(@NonNull Context context) {
        mContentResolver = context.getContentResolver();
    }
注意：在构造方法注入的时候这个类成为依赖图表的一部分。当其他类需要该这个类的对象时，dagger2会调用该构造方法创造其实例对象，该类的对象可以被注入其他对象中，如上部分的图所示，B就是通过构造器注入的方式生成，Component也可以根据需要将其注入到其他类中。
这两种注入方法中，需要的被注入的对象都从依赖图中获取，而不需要手工控制。
而对于Android中的一些组件，如Activity/Service/Fragment等等，由于其构造方法不由我们控制，只能通过域注入来完成。
5.Component：
Component就是我们前文提到的注入器(容器)，它定义了依赖图，并负责向其他类中注入依赖。
和Android的许多框架一样，对于Component部分，是有开发者以接口形式写Component，框架通过gradle插件自动生成Component的实例。形式有些像retrofit，但retrofit是通过动态代理，通过代理对象来完成接口任务的。
在实际开发中，我们定义的Component代码类似如下：
@Singleton
@Component(modules = {ApplicationModule.class, NumberInfoModule.class})
public interface AppComponent {

    void inject(MyService service);
    void inject(MyContentProvider p  rovider);

    @Component.Builder
    interface Builder {
        @BindsInstance
        Builder application(Application application);
        AppComponent build();
    }
}
上面例子中，我们定义了一个接口AppComponent，该接口被@Component注释，表明这是一个dagger2中的Component，Dagger2会在编译阶段生成其实例类:DaggerXXXComponent，以这个为例就是生成DaggerAppComponent，其实现了APPComponent。
我们来看这个接口的各个部分：
（1）@Singleton注解：这是一个scope的注释，dagger2的scope机制是为单利提供了生命周期的概念。@Singleton这表明了中依赖t的生命长度和Application生命长度一致。另外，@Singleton是JSR-330标准中的一部分，另外我们还可以自定义scope。有关scope的概念会在下一篇文章中介绍。
（2） Module：在上面@Component中我们指定了Module类，Module是为依赖图提供具体依赖的对象的，也就是我们在Module中我们告诉Dagger2框架当我们需要某个类的对象时，我们该如何获得。再回到最上面的图，其中A和C都是由Module提供的。也就是除了构造方法注入的类以外，其他的依赖需要有Module提供。后面会详细介绍。
（3）inject方法：Component在接口中，我们定义了inject的方法。在前面介绍通过域注入时，我们在组件的onCreate()方法中调用了Component的inject方法。该方法表明了我们可以将依赖图中的依赖注入到什么类中，该类(组件)在适当的时候（如onCreate()）调用inject方法，完成依赖注入。inject方法由我们写接口的时候定义，编译阶段框架实现该方法。
（4）Builder： 在Component接口中定义Component.Builder接口，顾名思义是在定义Component的建造者。上例中，用@BindsInstance
在定义Builder时候我们可以在允许Component初始化的时候设置一些对象，如上面例子，可以给Component设置Application对象，从而将Application对象纳入到依赖图中。
另外，如果其Module的构造函数需要传入参数的话，会自动生成Component.Builder设置该Module的方法，可以参考如下例子（也是一个添加Apllication依赖的实例）：
@Module
public class AppModule {
    Application mApplication;
    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    Application providesApplication() {
        return mApplication;
    }
}
@Singleton
@Component(modules={AppModule.class, NetModule.class})
public interface NetComponent {
   void inject(MainActivity activity);
   // void inject(MyFragment fragment);
   // void inject(MyService service);
}
实例化Component时可调用：
   mNetComponent = DaggerNetComponent.builder()
                .appModule(new AppModule(this)) // This also corresponds to the name of your module: %component_name%Module
                .build();
至此，我们可见Component维护了依赖图，而其中的依赖来源有：构造方法注入的对象，Component.Builder的@BindsInstance，以及Module类中提供的依赖。
6.Module
上文提到了，Module类是为依赖图中提供依赖的。一般从构造方法提供的依赖都有明确可调用的构造器才能够注入，而没有构造方法，例如从一些静态方法等方式获取的依赖就需要在Module中定义了。另外还有一种情况，如果我们想注入的是接口或者是父类，而注入的是接口的实际实现或其子类，也需要在Module中定义。下面通过Module中使用的一些注解来进行解释。
@Provides
@Provides注解允许我们在Module里定义方法，方法传入的参数是依赖图中已存在的依赖对象，返回将是该方法提供给依赖图的依赖。@Provides适用于需要由静态方法提供的依赖的情况,如:
@Module
class AppModule {
    @Singleton @Provides
    GithubService provideGithubService() {
        return new Retrofit.Builder()
                .baseUrl("https://api.github.com/")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(new LiveDataCallAdapterFactory())
                .build()
                .create(GithubService.class);
    }
  ...
}
另外需要通过调用依赖图中对象的某些方法才能获得的依赖
比如依赖图中有Context对象，例如我们希望在依赖图中或者ContentResolver对象，就可以定义方法：
@Module
class AppModule {
@Provides
    ContentProvider provideUserDao(Context context) {
        return context.getContentResolver();
    }
  ...
当然，还有其他情况下需要用到Provides注解，可以根据项目实际情况判断，就不举例了。
@Binds
@Binds和@Provides最大区别就是@Binds只能修饰抽象方法，比如说当A1类继承自A，而在当前的依赖图中可以提供A1的对象(如A1已经可以通过构造方法注入到Component中)，而在被注入类中需要A的对象，那么就可以定义Bind的抽象方法来将A1作为A的对象注入。再以上面AppComponent为例，Component实例化中通过Builder可以获得Application的对象，而如果依赖图中需要context，就可以提供给他们这个Application对象：
public abstract class ApplicationModule {
    //expose Application as an injectable context
    @Binds
    abstract Context bindContext(Application application);
}
所以@Binds解决了我们面向接口编程的需求。
当然这种情况也可以用@Provides的有实体方法(方法实体是类型的强转)，但@Binds明显更加清晰。
@Qualifier
而更进一步，如果依赖图中有两个子类都实现了某一接口，而我们在注入时在不同的场景下需要用这两个的某个，该怎么做呢？这时候我们需要@Qualifier的注解。
下面是一个实际的例子：
我们首先要定义两个 InfoRepository和RemoteInfoSource都是数据源，都实现了InfoSource接口， 分别表示本地数据源和云端的数据源（这种封装方式在MVP/MVVM架构中非常常见）。为区分二者，我们首先要先定义两个注释，@Remote表示远端数据，@Repository表示本地仓库：
@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Remote {}
@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Repository {}
注意，这两个Annotation都需要被@Qualifier修饰，@Qualifier为JSR-330标准中的一部分。
之后，InfoRepository和RemoteInfoSource在通过构造方法注入的时候，各自的构造方法在除了@Inject注释外还要加上刚刚定义的对应的Annotation：
public class InfoRepository {
    Context mContext;

    @Repository
    @Inject
    InfoRepository(@NonNull Context context) {
        mContext = context;
    }
public class RemoteInfoSource {
    Context mContext;

    @Remote
    @Inject
    RemoteInfoSource(@NonNull Context context) {
        mContext = context;
    }
接下来在Module中定义对应的@bind的抽象方法，对应方法也需要加对应Annotation：
@Module
abstract public class NumberInfoModule {

    @Binds @Repository
    abstract InfoSource provideLocalDataSource(InfoRepository dataSource);

    @Binds @Remote
    abstract InfoSource provideRemoteDataSource(RemoteInfoSource dataSource);
}
在使用时，如果我们需要Remote的InfoSource时就使用：
@Inject
@Remote
InfoSource mRemoteInfoSource;
或在构造方法注入时：
public Class TestClass {
  @Inject
  public TestClass(@Remote InfoSource cloudSource) {
    this.mRemoteInfoSource= remoteInfoSource;
  }
}
另外，@Qualifier 定义的注解可以设置参数，来标记不同的对象，如：
@Qualifier  
@Documented  
@Retention(RetentionPolicy.RUNTIME)
public @interface Source {
    String source() default "local";
}
就可以用@Source(soucrce = "remote")和@@Source(soucrce = "repository")代替上例中的@Remote和@Repository标签。
另外，@Named标签是JSR-330自带的一个@Qualifier实现，我们可以直接用@Named来起到和自定义@Qualifie注解r相同的效果，对应上面例子，@Remote和@Repository的位置替换成@Named("Remote")和@Named("Repository")，通过参数来区分同样类型的不同对象。
而需要本地仓库时，对应注解换成@Repository。
注意，在@Provides注解的方法中，同样可以用@Qualifier的标签。
小结
这篇文章首先介绍了何为控制反转和依赖注入，并介绍java依赖注入标准：JSR-330。随后，本文介绍了通过Dagger2构建依赖图的具体结构，着重介绍了如何将依赖注入到各个模块中，将对象增加到依赖图中的方法。介绍了Component的作用以及接口的定义方式。最后介绍了Module同@Provides和@Binds向依赖图提供依赖，并利用@Qualifier限定注入的对象。
我们可以总结出有三种方式想依赖图提供依赖：通过Component.Builder()的@BindInstance方式，通过构造方法注入的对象以及有Module的@Provides和@Binds提供的依赖。
Dagger2在Android应用中，最简单的情况是在Application完成Component的初始化，并Application中用静态方法向外提供Component实例，以让其他组件通过Component完成依赖注入。
但更复杂的情况下，还需要不同生命周期的Component来控制不同依赖图的生命，这就需要用到Scope以及Component间的依赖，以及Subcomponents，这些也都是Dagger2的重要概念，这将在下篇文章进行介绍。


















在上一篇文章(http://www.jianshu.com/p/f56d5b7e8b4d)中，我们接触到了Dagger2的基本用法。然而在实际Android开发当中，还会有更进一步的需求：需要有多个不同生命周期的多个Component，例如，有些依赖图是全局单例的，而有些依赖图会与Activity/Fragment同周期，或者有些依赖图是要与用户登录同周期。在这些情况下，我们就需要自定义Scope来维护多个Component各自依赖图的生命周期。更进一步的，我们讨论利用Component间的依赖和subComponent两种方法来创建多个Component。
1. Scope
Scope注解是JSR-330标准中的，该注解是用来修饰其他注解的。 前篇文章提到的@Singleton就是一个被Scope标注的注解，是Scope的一个默认实现。
Scope的注解的作用，是在一个Component的作用域中，依赖为单例的。也就是说同一个Component对象向各个类中注入依赖时候，注入的是同一个对象，而并非每次都new一个对象。
在这里，我们再介绍自定义的Scope注解，如：
@Scope
public @interface ActivityScope {
}
如上，ActivityScope就是一个我们自己定义的Scope注解，其使用方式与上篇文章中我们用Singleton的用法类似的。顾名思义，在实际应用中Singleton代表了全局的单例，而我们定义ActivityScope代表了在Activity生命周期中相关依赖是单例的。
Scope的注解具体用法如下：
(1). 首先用其修饰Component
(2). 如果依赖由Module的Provides或Binds方法提供，且该依赖在此Component中为单例的，则用Scope相关注解修饰Module的Provides和Binds方法。
(3). 如果依赖通过构造方式注入，且该依赖在此Component中为单例的，则要用Scope修饰其类。
我们通过如下例子详细说明，并进行测试和分析其原理：
以Singleton这个Scope是实现为例：
首先用它来修饰Component类：
@Singleton
@Component
public interface AppComponent {
    void inject(MainActivity mainActivity);
}
用通过构造方法注入依赖图的，用@Singleton修饰其类：
@Singleton
public class InfoRepository {
    private static final String TAG = "InfoRepository";

    @Inject
    InfoRepository() {
    }

    public void test() {
        Log.d(TAG, "test");
    }
}
实际注入到Activity类中如下：
public class MainActivity extends Activity {
    @Inject
    InfoRepository infoRepositoryA;
    @Inject
    InfoRepository infoRepositoryB;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        MyApplication.getComponent().inject(this);
        setContentView(R.layout.activity_main);
        Log.d("test", "a:"+infoRepositoryA.hashCode());
        Log.d("test", "b:"+infoRepositoryB.hashCode());
    }
}
如上代码测试结果如下：
10-19 19:25:08.253 26699-26699/com.qt.daggerTest D/test: a:442589
10-19 19:25:08.254 26699-26699/com.qt.daggerTest D/test: b:442589
如上可见，两次注入的InfoRepository对象为同一个。
如果将上面修饰InfoRepository的@Singleton注解去掉，结果是什么呢？经过测试如下：
10-19 19:23:00.092 23014-23014/com.qt.daggerTest D/test: a:442589
10-19 19:23:00.092 23014-23014/com.qt.daggerTest D/test: b:160539730
也就是说在不加@Singleton注解时候，每次注入的时候都是new一个新的对象。
注意，如果我们将上面所有的@Singleton替换成我们自己的Scope注解其结果也是一致的，如替换成上文的ActivityScope。
下面，我们通过分析Dagger自动生成的代码来分析如何实现单例的：
在注入类不加@Singleton注解时，生成的DaggerAppComponent类的initialize()方法如下：
  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {
    this.mainActivityMembersInjector =
        MainActivity_MembersInjector.create(InfoRepository_Factory.create());
  }
而加@Singleton注解后的initialize()方法变成了：
  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.infoRepositoryProvider = DoubleCheck.provider(InfoRepository_Factory.create());

    this.mainActivityMembersInjector =
        MainActivity_MembersInjector.create(infoRepositoryProvider);
  }
也是就是说提供InfoRepository的InfoRepositoryProvider替换成了DoubleCheck.provider(InfoRepository_Factory.create())。用DoubleCheck包装了原来对象的Provider。DoubleCheck顾名思义，应该是通过双重检查实现单例，我们看源码确实如此：
public final class DoubleCheck<T> implements Provider<T>, Lazy<T> {
  private static final Object UNINITIALIZED = new Object();

  private volatile Provider<T> provider;
  private volatile Object instance = UNINITIALIZED;

  private DoubleCheck(Provider<T> provider) {
    assert provider != null;
    this.provider = provider;
  }

  @SuppressWarnings("unchecked") // cast only happens when result comes from the provider
  @Override
  public T get() {
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          result = provider.get();
          /* Get the current instance and test to see if the call to provider.get() has resulted
           * in a recursive call.  If it returns the same instance, we'll allow it, but if the
           * instances differ, throw. */
          Object currentInstance = instance;
          if (currentInstance != UNINITIALIZED && currentInstance != result) {
            throw new IllegalStateException("Scoped provider was invoked recursively returning "
                + "different results: " + currentInstance + " & " + result + ". This is likely "
                + "due to a circular dependency.");
          }
          instance = result;
          /* Null out the reference to the provider. We are never going to need it again, so we
           * can make it eligible for GC. */
          provider = null;
        }
      }
    }
    return (T) result;
  ...
  }
其get方法就用了DoubleCheck方式保证了单例，其中还判断如果存在循环依赖的情况下抛出异常。
注意，Scope只的单例是在Component的生命周期中相关依赖是单例的，也就是同一个Component对象注入的同类型的依赖是相同的。按上面例子，如果我们又创建了个AppComponent，它注入的InfoRepository对象与之前的肯定不是一个。
2. Component间依赖
在Android应用中，如果我们需要不止一个Component，而Component的依赖图中有需要其他Component依赖图中的某些依赖时候，利用Component间依赖（Component Dependency）方式是个很好的选择。在新建Component类时候可以在@Component注解里设置dependencies属性，确定其依赖的Component。在被依赖的Component中，如果要暴露其依赖图中的某个依赖给其他Component，要显示的在其中定义方法，使该依赖对其他Component可见如：
@Singleton
@Component(modules = AppModule.class)
public interface AppComponent {
    Application application();
}
@ActivityScope
@Component(dependencies = AppComponent.class, modules = ActivityModule.class)
public interface ActivityComponent {
    void inject(MainActivity mainActivity);
}
在ActivityComponent中，就可以使用AppComponent依赖图中暴露出的Application依赖了。
非常清晰，Component依赖(Component Dependency)的方式适用于Component只将某个或某几个依赖暴露给其他Component的情况下。
3.subComponent
定义subComponent是另一种方式使Component将依赖暴露给其他Component。当我们需要一个Component，它需要继承另外一个Component的所有依赖时，我们可以定义其为subComponent。
具体用法如下：首先在父Component的接口中定义方法，来获得可以继承它的subComponent：
@Singleton
@Component(
        modules = {AppModule.class}
)
public interface AppComponent {

    UserComponent plus(UserModule userModule);

}
其次，其subComponent被@SubComponent注解修饰，如下：
@UserScope
@Subcomponent(
        modules = {UserModule.class}
)
public interface UserComponent {
  ...
}
Dagger会实现上面在接口中定义的plus()方法，我们通过调用父Component的plus方法或者对应的subComponent实例，具体如下：
userComponent = appComponent.plus(new UserModule(user));
这样，我们就获得了userComponent对象，可以利用他为其他类注入依赖了。
注意，subComponent继承了其父Component的所有依赖图，也就是说被subComponent可以向其他类注入其父Component的所有依赖。
4. 用多个Component组织你的Android应用
前面几部分中，我们介绍了如何创建多个Component/subComponent并使其获得其他Component的依赖。这就为我们在应用中用多个Component组织组织应用提供了条件。文章http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/ 提供了一个常用的思路：我们大概需要三个Component：AppComponent，UserComponent，ActivityComponent，如下：


摘自http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/

上文介绍了，各个Component要有自己的Scope且不能相同，所以这几个Component对应的Scope分别为@Singleton ，@UserScop，@ActivityScope。也就是说，依赖在对应的Component生命周期(同个Component中)中都是单例的。而三个Component的生命周期都不同：AppComponent为应用全局单例的，UserComponent的生命周期对应了用户登录的生命周期(从用户登录一个账户到用户退出登录)，ActivityComponent对应了每个Activity的生命周期，如下：



Scopes lifecycle 摘自http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/

在具体代码中，我们通过控制Component对象的生命周期来控制依赖图的周期，以UserComponent为例，在用户登录时候创建UserComponent实例，期间一直用该实例为相关类注入依赖，当其退出时候将UserComponent实例设为空，下次登录时候重新创建个UserComponent，大致如下：
    public class MyApplication extends Application {
      ...
    private void initAppComponent() {
        appComponent = DaggerAppComponent.builder()
                .appModule(new AppModule(this))
                .build();
    }

    public UserComponent createUserComponent(User user) {
        userComponent = appComponent.plus(new UserModule(user));
        return userComponent;
    }

    public void releaseUserComponent() {
        userComponent = null;
    }

    public AppComponent getAppComponent() {
        return appComponent;
    }

    public UserComponent getUserComponent() {
        return userComponent;
    }

}
createUserComponent和releaseUserComponent在用户登入和登出时候调用，所以在不同用户中用的是不同UserComponent对象注入，注入的依赖也不同。而AppComponent对象只有一个，所以其依赖图中的依赖为全局单例的。而对于ActivityComponent，则可以在Activity的onCreate()中生成ActivityComponent对象来为之注入依赖。
5.多Component情况下Scope的使用限制
Scope和多个Component在具体使用时候有一下几点限制需要注意：
(1). Component和他所依赖的Component不能公用相同的Scope，每个Component都要有自己的Scope，编译时会报错，因为这有可能破坏Scope的范围，可详见https://github.com/google/dagger/issues/107#issuecomment-71073298。这种情况下编译会报错：
Error:(21, 1) 错误:com.qt.daggerTest.AppComponent depends on scoped components in a non-hierarchical scope ordering:
@com.qt.daggerTest.ActivityScope com.qt.daggerTest.AppComponent
@com.qt.daggerTest.ActivityScope com.qt.daggerTest.ActivityComponent
(2). @Singleton的Component不能依赖其他Component。这从意义和规范上也是说的通的，我们希望Singleton的Component应为全局的Component。这种情况下编译时会报错：
Error:(23, 1) 错误: This @Singleton component cannot depend on scoped components:
@Singleton com.qt.daggerTest.AppComponent
(3). 无Scope的Component不能依赖有Scope的Component，因为这也会导致Scope被破坏。这时候编译时会报错：
Error:(20, 2) 错误: com.qt.daggerTest.ActivityComponent (unscoped) cannot depend on scoped components:
@com.qt.daggerTest.ActivityScope com.qt.daggerTest.AppComponent
(4). Module以及通过构造函数注入依赖图的类和其Component不可有不相同的Scope，这种情况下编译时会报：
Error:(6, 1) 错误: com.qt.daggerTest.AppComponent scoped with @com.qt.daggerTest.ActivityScope may not reference bindings with different scopes:
@Singleton class com.qt.daggerTest.InfoRepository
总结
在上一篇Dagger2介绍与使用(http://www.jianshu.com/p/f56d5b7e8b4d)的基础之上，本文围绕着Android中多个Component情况下如何使用的问题展开了分析。首先说明了如何通过Scope和Component管理依赖的生命周期，再介绍了通过Component间依赖和subComponent两种方式完成一个Component将自己的依赖图暴露给其他Component的过程。然后介绍如一般在Android应用中如何划分Component，维护不同生命周期的依赖。
经过两篇文章的介绍，Dagger2的使用基本清晰。Dagger2在处理Android应用的依赖非常得心应手，通过依赖注入的方式实现依赖反转，用容器(Component)来控制了所有依赖，使应用各个组件的依赖更加清晰。Dagger2的各种功能也比较灵活，能够应付Android应用依赖关系的各种复杂场景。


