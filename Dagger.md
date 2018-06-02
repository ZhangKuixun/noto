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


作者：qintong000
链接：https://www.jianshu.com/p/f56d5b7e8b4d
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。