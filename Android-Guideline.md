#  **Android编码规范**

## **使用Android studio作为集成开发环境**

对于编辑器，每个人都有自己的选择，让编辑器根据工程结构和构建系统高效运作，是每个人的责任。

推荐使用[Android Studio](https://developer.android.com/sdk/installing/studio.html)，由谷歌开发，并且最接近**Gradle**，默认使用最新的工程结构。[Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt)使用旧的工程结构和Ant作为构建系统，它不仅需要繁琐的配置，而且`gardle`和`adb`命令行同样需要学习成本。


## **使用Gradle构建项目**

默认编译环境应该使用[Gradle](http://tools.android.com/tech-docs/new-build-system)。Ant不仅有限制而且操作方式非常繁琐，使用Gradle编译，可以轻松实现以下几点：

1. 构建App的不同版本，在debug和release之间轻松切换。
2. 快速制作简单的脚本任务
3. 轻松下载和管理依赖库
4. 能够方便的按照要求定制Keystore


## **项目结构**

废弃过时的**Ant & Eclipse ADT**工程结构，统一使用新的**Gradle & Android Studio**的工程结。

旧式Eclipse结构：

```
old-link-structure
├─ assets
├─ libs
├─ res
├─ src
│  └─ com/im/project
├─ AndroidManifest.xml
├─ build.gradle
├─ project.properties
└─ proguard-rules.pro
```

新式Android studio结构：

```
new-link-structure
├─ library-imsdk
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ com/im/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/im/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```

新的项目结构更加分明，强调了**Gradle**概念。其中`library-imsdk`是`app`所依赖的module。


## **签名配置**

发布release版本的时候，应该做好**SigningConfigs**的保密性：

不要这样做，这种做法非常的不安全：

```groovy
signingConfigs {
	release {
		storeFile file("myapp.keystore")
		storePassword "storePassword"
		keyAlias "storeKey"
		keyPassword "keyPassword"
	}
}
```
而是，建立一个不加入版本控制系统的`gradle.properties`文件，或者在本地的`local.properties`中记录。

```
KEYSTORE_PASSWORD=storePassword
KEY_PASSWORD=keyPassword
```

那个文件会被gradle自动引入，因此可以在`buld.gradle`文件中使用，例如：

```groovy
signingConfigs {
	release {
		try {
			storeFile file("myapp.keystore")
			storePassword KEYSTORE_PASSWORD
			keyAlias "storeKey"
			keyPassword KEY_PASSWORD
		}
		catch (ex) {
			throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
		}
	}
}
```

## **使用Maven依赖代替jar包导入**

如果在项目中明确使用jar文件，那么它们可能成为永久的版本，如`2.1.1`。而且很容易陷入频繁更新jar包的繁琐工作中，Maven很好的解决了这个问题，这也是 Gradle构建中推荐的方法。你可以指定版本的一个范围，如`2.1.+`,然后Maven会自动升级到制定的最新版本，例如：

```groovy
dependencies {
      /*ReactiveX Library*/
      compile 'io.reactivex:rxjava:1.1.3'
      compile 'io.reactivex:rxandroid:1.1.0'
}
```

## **命名规范**

### **Java文件**

对Java类文件使用[驼峰命名法](https://en.wikipedia.org/wiki/CamelCase)。

对于继承自Android组件的类来说，类名应该以这个组件的类型作后缀，如：`SignInActivity, UserProfileFragment, ImageUploaderService, ChangePasswordDialog`

### **XML文件**

资源等`.xml`文件应该采用**小写字母_下划线**的组合形式。

#### **Drawable相关**

- 常规Drawable（图像）文件命名方式：

| Asset Type   | Prefix            |		Example              |
|--------------| ------------------|-----------------------------|
| Action bar   | `ab_`             | `ab_stacked.9.png`          |
| Button       | `btn_`	           | `btn_send_pressed.9.png`    |
| Dialog       | `dialog_`         | `dialog_top.9.png`          |
| Divider      | `divider_`        | `divider_horizontal.9.png`  | 
| Icon         | `ic_`	           | `ic_star.png`               |
| Menu         | `menu_	`          | `menu_submenu_bg.9.png`     |
| Notification | `notification_`   | `notification_bg.9.png`     |
| Tabs         | `tab_`            | `tab_pressed.9.png`         |


- 常规icon（图标）文件命名方式：

| Asset Type                      | Prefix             | Example                      |
| --------------------------------| ----------------   | ---------------------------- |
| Icons                           | `ic_`              | `ic_star.png`                |
| Launcher icons                  | `ic_launcher`      | `ic_launcher_calendar.png`   |
| Menu icons and Action Bar icons | `ic_menu`          | `ic_menu_archive.png`        |
| Status bar icons                | `ic_stat_notify`   | `ic_stat_notify_msg.png`     |
| Tab icons                       | `ic_tab`           | `ic_tab_recent.png`          |
| Dialog icons                    | `ic_dialog`        | `ic_dialog_info.png`         |


- 常规selector states（选中状态）文件命名方式：

| State	       | Suffix          | Example                     |
|--------------|-----------------|-----------------------------|
| Normal       | `_normal`       | `btn_order_normal.9.png`    |
| Pressed      | `_pressed`      | `btn_order_pressed.9.png`   |
| Focused      | `_focused`      | `btn_order_focused.9.png`   |
| Disabled     | `_disabled`     | `btn_order_disabled.9.png`  |
| Selected     | `_selected`     | `btn_order_selected.9.png`  |


#### **Lyout相关**

- 布局（Layout）文件命名方式：

布局文件应该与Android组件的命名相匹配，以组件类型作为前缀，并且能够清晰的表达意图所在。例如：如果为`SignInActivity`创建一个布局文件，那么应该命名为`activity_sign_in.xml`。基本规则如下：

| Component        | Class Name             | Layout Name                   |
| ---------------- | ---------------------- | ----------------------------- |
| Activity         | `UserProfileActivity`  | `activity_user_profile.xml`   |
| Fragment         | `SignUpFragment`       | `fragment_sign_up.xml`        |
| Dialog           | `ChangePasswordDialog` | `dialog_change_password.xml`  |
| AdapterView item | ---                    | `item_person.xml`             |
| Partial layout   | ---                    | `partial_stats_bar.xml`       |

值得一提的是，一些布局文件需要通过`Adapter`填充，如`ListView`，`Recyclerview`等列表视图，这种场景下，布局的命名应该以`item_`作为前缀。另外还有一种比较常见的情况，一个布局文件作为另一个布局文件的一部分而存在，或者使用了`include`，`merge`等标签的布局，可以使用`partial_`、`include_`或者`merge_`作为前缀，这一类布局的命名同样应该清晰的表达其意图。

- Id命名方式：

控件Id的命名应该以该控件类型的缩写作为后缀，并且也应该是**小写字母_下划线**的组合形式，能够清晰表达意图是命名的前提：

| Element            | Suffix          |
| -----------------  | ----------------|
| `TextView`         | `_tv`           |
| `ImageView`        | `_iv`           |
| `Button`           | `_btn`          |
| `Menu`             | `_menu`         |

例如：

```xml
<ImageView
    android:id="@+id/profile_iv"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

```xml
<menu>
	<item
        android:id="@+id/done_menu"
        android:title="Done" />
</menu>
```


#### **Color相关**

`colors.xml`文件就像个“调色板”，只映射颜色的ARGB值，不应该存在其他类型的数值，更不要使用它为不同的按钮来定义ARGB值。

不建议使用以下命名规则：

```xml
<resources>
	<color name="button_foreground">#FFFFFF</color>
	<color name="button_background">#2A91BD</color>
	<color name="comment_background_inactive">#5F5F5F</color>
	<color name="comment_background_active">#939393</color>
	<color name="comment_foreground">#FFFFFF</color>
	<color name="comment_foreground_important">#FF9D2F</color>
	...
	<color name="comment_shadow">#323232</color>
```

使用这种定义方式，我们需要非常的谨慎，一不小心就会重复定义ARGB值，而且当改变基本色时，会造成很多冗余重复的操作。

相反地，我们应该根据颜色或者风格对ARGB赋值：

```xml
<resources>

	<!-- grayscale -->
	<color name="white"     >#FFFFFF</color>
	<color name="gray_light">#DBDBDB</color>
	<color name="gray"      >#939393</color>
	<color name="gray_dark" >#5F5F5F</color>
	<color name="black"     >#323232</color>

	<!-- basic colors -->
	<color name="green"     >#27D34D</color>
	<color name="blue"      >#2A91BD</color>
	<color name="orange"    >#FF9D2F</color>
	<color name="red"       >#FF432F</color>

</resources>
```

对同一色调，不同色域进行定义时，像"brand_primary"、"brand_secondary"、 "brand_negative"这样的命名也是不错的选择。

值得一提的是，这样规范的颜色很容易修改或重构，App一共使用了多少种不同的颜色变会得非常清晰。


#### **Dimen相关**

我们应该像对待`colors.xml`一样对待`dimens.xml`文件，与定义颜色调色板无异，也应该定义一个规范字体大小的“字号板”。

一个很好的建议：

```xml
<resources>

	<!-- font sizes -->
	<dimen name="font_larger">22sp</dimen>
	<dimen name="font_large">18sp</dimen>
	<dimen name="font_normal">15sp</dimen>
	<dimen name="font_small">12sp</dimen>

	<!-- typical spacing between two views -->
	<dimen name="spacing_huge">40dp</dimen>
	<dimen name="spacing_large">24dp</dimen>
	<dimen name="spacing_normal">14dp</dimen>
	<dimen name="spacing_small">10dp</dimen>
	<dimen name="spacing_tiny">4dp</dimen>

	<!-- typical sizes of views -->
	<dimen name="button_height_tall">60dp</dimen>
	<dimen name="button_height_normal">40dp</dimen>
	<dimen name="button_height_short">32dp</dimen>

</resources>
```

同样的，在定义`margin`和`padding`时，可以使用`spacing_****`作为前缀对其命名，而不是像对待`String`字符串那样直接写值。

这样写的好处是，使组织结构和修改风格甚至布局变得非常容易。

#### **String相关**

String命名的前缀应该能够清楚地表达它的功能职责，如，`registration_email_hint`，`registration_name_hint`。如果一个Sting不属于任何模块，这也就意味着它是通用的，应该遵循以下规范：


| Prefix             | Description                           |
| -----------------  | --------------------------------------|
| `error_`           | 错误提示                              |
| `msg_`             | 一般信息提示                          |
| `title_`           | 标题提示，如，Dialog标题              |
| `action_`          | 动作提示，如，“保存”，“取消”，“创建”  |

#### **Style与Theme相关**

Style与Theme的命名统一使用[驼峰命名法](https://en.wikipedia.org/wiki/CamelCase)。应该谨慎使用`style`与`theme`，避免重复冗余的文件出现。可以有多个`styles.xml` 文件，如：`styles.xml`，`style_home.xml`，`style_item_details.xml`，`styles_forms.xml`等。
**`res/values`目录下的文件可以任意命名，但前提是该文件能够明确表达职责所属，因为起作用的并不是文件本身，而是内部的标签属性。**


## **编码规范与指导方针**

### **XML文件规范**

#### **属性排序**

对于如何排版一个布局文件，请尽量遵循以下规范：

- 每个属性独占一行，缩进四个空格
-  `android:id`作为第一个属性存在
- 如果存在`style`属性，则紧随`id`之后
- 如果不存在`style`属性，则`android:layout_****`紧随`id`之后
- 当布局中的一个元素不再包含子元素时，另起一行，使用自闭合标签`/>`，方便调整和添加新的属性

示例如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/name"
        style="@style/FancyText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:text="@string/name"
        />

    <include layout="@layout/reusable_part" />

</LinearLayout>
```

#### **使用`tools`标签预览视图**

- 布局预览应使用`tools:****`相关属性，避免`android:text`等硬编码的出现，具体可参考[Designtime attributes](http://tools.android.com/tips/layout-designtime-attributes)。
示例如下：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="Home Link" />
```

避免以下代码的出现：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Home Link" />
```

#### **通用`style`**

值得一提的是，`android:layout_****`属性应该在XML中定义，同时其它属性`android:****`应放在`style`中。核心准则是保证Layout属性(position, margin, size等)和content属性在布局文件中，同时将所有的外观细节属性（color, padding, font）放
在style文件中。

另外，在上面提到的准则中，有以下几点需要注意：

- `android:id`明显应该在layout文件中
- Layout文件中的`android:orientation`属性对于一个`LinearLayout`布局来说，更具有意义
- 由于使用`android:text`定义内容，所以这个属性应该放在Layout文件中
- 有时候将`android:layout_width`和`android:layout_height`属性放到一个`style.xml`中作为一个通用的风格更有意义，但是默认情况下把这些属性放到Layout文件中比放到`style.xml`文件中更加直观。

#### **避免层级冗余的嵌套**

Layout结构优化方面，应尽量避免深层次的布局嵌套，这不仅会引发性能瓶颈，还会带来项目维护上的麻烦。在书写布局之前应该对ViewTree充分的分析，善用[`<merge>`标签](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts)减少层级嵌套，或者使用[Hierarchy Viewer](http://developer.android.com/intl/zh-cn/tools/help/hierarchy-viewer.html)等UI优化工具对Layout进行分析与优化。可参考[Optimizing Your UI](http://developer.android.com/intl/zh-cn/tools/debugging/debugging-ui.html)与[Optimizing Layout Hierarchies](http://developer.android.com/intl/zh-cn/training/improving-layouts/optimizing-layout.html)。


### **Java代码规范**

#### **处理`Exception`**

如果对异常没有任何处理方案，至少应该在`catch`代码块中添加打印异常逻辑`e.printStackTrace()`，以下是不建议采用的处理方式：

```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```

你可能认为这种异常的触发概率极低，或者认为即便发生，也不需要做着重的处理，因此在`catch()`代码块中不添加任何逻辑，这大大增加了定位异常的难度，这里的建议是尽量处理每一个异常，具体的处理逻辑依赖于所处的上下文场景，对异常的处理同样能够增加系统的弹性，让程序变更健壮。

#### **Field定义与命名规范**

对Field的定义应该放在文件的首位，并且遵守以下规范：

- 被Private修饰的非静态变量，以**m**作为前缀
- 被Private修饰的静态变量，以**s**作为前缀
- 其他变量以小写字母做前缀
- 静态常量命名字母全部大写，单词之间用下划线分隔

示例：

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```

#### **缩写词命名规范**

| Good             | Bad              |
| --------------   | --------------   |
| `XmlHttpRequest` | `XMLHTTPRequest` |
| `getCustomerId`  | `getCustomerID`  |
| `String url`     | `String URL`     |
| `long id`        | `long ID`        |


#### **注解使用规范**

- `@Override`： 子类实现或者重写父类方法时，必须使用`@Override`对函数进行标注。
- `@SuppressWarnings`： 注解`@SuppressWarnings`应该用在消除那些明确不可能发生的警告上，示例如下：

```java
//明确的类型安全（type-safe）转换
@SuppressWarnings("unchecked")
public static <R> FeedbackUseCase<R> createdUseCase() {
    return (FeedbackUseCase<R>) new FeedbackUseCase();
}

//请先确保能够非常正确地使用Handler
@SuppressWarnings("handlerLeak")
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
    }
};
```
更多关于注解的使用技巧与规范请参考[这里](http://source.android.com/source/code-style.html#use-standard-java-annotations)。

#### **限制变量的作用域**

定义变量时，应该保证变量所在域的最小化，这样做不仅能够增加代码的可读性和可维护性，还能有效减少出错的可能性。变量定义在尽可能小的域中，也是良好封装性的体现。

变量应该在首次使用时被初始化，如果没有足够的条件来使用它，那么就应该延迟创建，直到真正需要使用的时候再去进行初始化操作。

#### **Log输出规范**

使用`Log`类打印一些重要的信息对开发者而言是很重要的事情，切记不要使用`Toast`来做信息打印。

针对不同的信息使用对应的日志输出类型：

- `Log.v(String tag, String msg)` (verbose)
- `Log.d(String tag, String msg)` (debug)
- `Log.i(String tag, String msg)` (information)
- `Log.w(String tag, String msg)` (warning)
- `Log.e(String tag, String msg)` (error)

一般情况下，当前类名作为`tag`，并且使用`static final`修饰，放在类文件的首位，示例如下：

```java
public class MyClass {
    private static final String TAG = MyClass.class.getSimpleName();

    public myMethod() {
        Log.e(TAG, "My error message");
    }
}
```

**VERBOSE**和**DEBUG**类型的Log不应该出现在Release版本中，**INFORMATION**、**WARNING**和**ERROR**类型的Log可以留下来，因为这些信息的输出能够帮助我们快速地定位问题所在，当然前提是，需要隐藏重要的信息输出，如，用户手机号，邮箱等。

只在Debug环境中输出日志的小技巧：

```java
if (BuildConfig.DEBUG) Log.d(TAG, "The value of x is " + x);
```

#### **类成员排序规范**

关于这个并没有硬性要求，不过好的排序方式，能够提高可读性和易学性。这里给出一些排序建议：

1. 常量
2. 字段
3. 构造函数
4. 被重写的函数（不区分修饰符类型）
5. 被`private`修饰的函数
6. 被`public`修饰的函数
7. 被定义的内部类或者接口

示例如下：

```
public class MainActivity extends Activity {
    
    private static final String TAG = MainActivity.class.getSimpleName();
	private String mTitle;
    private TextView mTextViewTitle;

    @Override
    public void onCreate() {
        ...
    }

    private void setUpView() {
        ...
    }
    
    public void setTitle(String title) {
        mTitle = title;
    }

    static class AnInnerClass {

    }
}
```

如果继承了Android组件，比如Activity或者Fragment，重写生命周期函数时，应该按照组件的生命周期进行排序，如：

```java
public class MainActivity extends Activity {

    @Override
    public void onCreate() {}

    @Override
    public void onResume() {}

    @Override
    public void onPause() {}

    @Override
    public void onDestroy() {}
    
}
```

#### **方法函数中的参数排序规范**

在Android日常开发中，很多情况下都需要使用`Context`，所以经常被作为参数传入方法中，这里给出的建议是，如果函数签名中存在`Context`，则作为第一个参数，如果存在`Callback`则作为最后一个参数，示例如下：

```java
public void loadUserAsync(Context context, int userId, UserCallback callback);
```

#### **字符串常量的命名与赋值规范**

Android SDK中诸如`SharedPreferences`，`Bundle`和`Intent`等，都采用**key-value**的方式进行赋值，当使用这些组件的时候，**key**必须被`static final`所修饰，并且命名应该符合以下规范：

| Element            | Field Name Prefix   |
| -----------------  | -----------------   |
| SharedPreferences  | `PREF_`             |
| Bundle             | `BUNDLE_`           |
| Fragment Arguments | `ARGUMENT_`         |
| Intent Extra       | `EXTRA_`            |
| Intent Action      | `ACTION_`           |


需要注意的是，当调用Fragment的`.getArguments()`方法时，返回值同样是一个`Bundle`，因为这也是一个常用函数，因此我们需要定义一个不同的前缀，示例如下：

```java
static final String PREF_EMAIL = "PREF_EMAIL";
static final String BUNDLE_AGE = "BUNDLE_AGE";
static final String ARGUMENT_USER_ID = "ARGUMENT_USER_ID";

static final String EXTRA_SURNAME = "com.myapp.extras.EXTRA_SURNAME";
static final String ACTION_OPEN_USER = "com.myapp.action.ACTION_OPEN_USER";
```

#### **Activity和Fragment打开方式**

当通过`Intent`或者`Bundle`向`Activity`与`Fragment`传值时，应该遵循上面提到的`key-value`规范，公开一个被`public static`修饰的方法，方法的参数应该包含所有打开这个`Activity`或者`Fragment`的信息，示例如下：

- 通过`.startActivity()`函数，开启指定**Activity**。

```java
public static void startActivity(AppCompatActivity startingActivity, User user) {
    Intent intent = new Intent(startingActivity, ThisActivity.class);
    intent.putParcelableExtra(EXTRA_USER, user);
    startingActivity.startActivity(intent);
  }
```

- 通过`.newInstance()`函数，加载指定`Fragment`。

```java
public static UserFragment newInstance(User user) {
	UserFragment fragment = new UserFragment;
	Bundle args = new Bundle();
	args.putParcelable(ARGUMENT_USER, user);
	fragment.setArguments(args)
	return fragment;
}
```

需要注意一下两点：

1. 以上这些方法应该放在类的开头，至少应该放在`.onCreate()`之前。
2. 如`EXTRA_USER`，`ARGUMENT_USER`等常量`key`，应该放在本类中被`private`所修饰，不应该暴露给其它外部类。


### **提高程序可读性**

我们应该在日常开发中养成添加注释的习惯，避免**魔鬼数字**与**硬编码**的出现，这样不仅可以提高可读性与可维护性，还能为逻辑的重构带来很大的便捷。

这里有个很好的建议，使用[RxJava](https://github.com/ReactiveX/RxJava)让程序变得更加可读，更加函数化。

这里有一个示例，可供参考，“对一个数组中的每一个元素乘以2，然后筛选小于10的元素，并放入最终的集合中”

一般写法示例：

```java
Integer[] integers = { 0, 1, 2, 3, 4, 5 };

Integer[] doubles = new Integer[integers.length];
for (int i = 0; i < integers.length; i++) {
    doubles[i] = integers[i] * 2;
}
List<Integer> results = new ArrayList<>(doubles.length);
for (Integer integer : doubles) {
    if (integer < 10) {
        results.add(integer);
    }
}
```

使用`RxJava`的流式调用示例，提高可读性：

```java
//假设你了解这些操作符的工作原理
List<Integer> results = Observable.from(integers)
                                  .map(new Func1<Integer, Integer>() {
                                      @Override
                                      public Integer call(Integer integer) {
                                          return integer * 2;
                                      }
                                  })
                                  .filter(new Func1<Integer, Boolean>() {
                                      @Override
                                      public Boolean call(Integer integer) {
                                          return integer < 10;
                                      }
                                  })
                                  .toList()
                                  .toBlocking()
                                  .first();
```

另外需要补充的一点是：当使用`Builder`或者其他链式结构创建对象时，每一个方法的调用都应另起一行，并且每一行都以`.`开头，示例如下：

```java
Picasso.with(context)
        .load(R.drawable.placeholder)
        .into(imageView);
```


## **Git分支管理与提交规范**

多人协作开发情况下，每个人都应该了解并理解**Git分支模型**

这里意见几篇质量极高的文章以供参阅，因此不再赘述。

[git版本控制开发流程小结笔记（一）](http://my.oschina.net/nyankosama/blog/270546?fromerr=0Byh13go)

[git版本控制开发流程小结笔记（二）](http://my.oschina.net/nyankosama/blog/270581)

[一个成功的 Git 分支模型](http://www.oschina.net/translate/a-successful-git-branching-model)


因为Git的操作指令非常灵活，所以这里只给出提交备注的建议：

**1. 提交备注**

- 如果使用类似`$ git commit`等语句，建议采用以下规范添加完整的提交备注

    标题与内容之间保持一个空行，示例如下：
    
            Fixed #<此bug在bug管理工具上的索引、路径或其他标识>
        
            内容部分添加简短描述，如改动原因，主要变动或者一些重要的建议事项。最后如果有需要可以添加对应的网址，如bug地址等。
            
- 如果使用`$ git commit -m""`等语句，建议采用以下规范添加简短的提交备注

    如果真的没有什么详细内容可以描述，使用简短的提交备注形式，是一种不错的选择，示例如下：
    
        git commit -m 'Fixed #[issue number]: [Short summary of the change].'
       
下面是几个常用提交动词，以供参考：

- Added ( 新加入的需求 )
- Removed （删除某段代码）
- Fixed ( 修复Bug，建议后面加上Bug索引或路径，如：Fixed #[issue number] )
- Updated ( 完成的任务，或者由于第三方模块变化而做的变化） 
- Completed ( 完成的任务 )
- Refactored（重构代码） 

**2.  标题尽量不超过50个字符**

**3. 标题的首字母大写（如果你习惯使用一个英文标题的话）**

    使用这种标题：

        Accelerate to 88 miles per hour
    
    而不是：

        accelerate to 88 miles per hour
        
**4. 标题不以“句号”作为结尾**
    
    使用这种标题：
    
        Open the pod bay doors

    而不是：
    
        Open the pod bay doors.
        
**5. 内容描述尽量不要超过72个字符**

**6. 在内容中对此次提交的“Why”以及“How”来进行描述**


最后，我们一起来看一看，一个优雅的提交备注是什么样子的：

       Simplify serialize.h's exception handling
    
       Remove the 'state' and 'exceptmask' from serialize.h's stream
       implementations, as well as related methods.
    
       As exceptmask always included 'failbit', and setstate was always
       called with bits = failbit, all it did was immediately raise an
       exception. Get rid of those variables, and replace the setstate
       with direct exception throwing (which also removes some dead
       code).
    
       As a result, good() is never reached after a failure (there are
       only 2 calls, one of which is in tests), and can just be replaced
       by !eof().
    
       fail(), clear(n) and exceptions() are just never called. Delete
       them.

