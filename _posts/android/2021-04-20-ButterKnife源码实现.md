---
layout:     post
title:      "ButterKnife源码实现"
date:       2021-04-20
author:     "zhouhaoh"
catalog: true
tags:
- Android
---

从butterKnife简单看apt技术的应用



### APT与IOC框架区别

1. 共同优点：实现了解耦目的。
2. 技术点：ioc利用运行时反射技术，apt是注解处理器技术
3. ioc编写反射难一点，apt需要对照代码模板生成代码。
4. 缺点：ioc在运行时反射会消耗一定的性能，apt由于生成了辅助类的代码文件，会增加apk包的大小



### 代码流程

#### 在activity中初始化

```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ButterKnife.bind(this);
    }
```

进入bind方法：

```java
 public static Unbinder bind(@NonNull Activity target) {
    View sourceView = target.getWindow().getDecorView();
    return createBinding(target, sourceView);
  }
```

调用createBinding方法：

```java
 private static Unbinder createBinding(@NonNull Object target, @NonNull View source) {
    Class<?> targetClass = target.getClass();
    if (debug) Log.d(TAG, "Looking up binding for " + targetClass.getName());
     //注释1 发现一个构造方法
    Constructor<? extends Unbinder> constructor = findBindingConstructorForClass(targetClass);

    if (constructor == null) {
      return Unbinder.EMPTY;
    }

    //noinspection TryWithIdenticalCatches Resolves to API 19+ only type.
    try {
        //注释2
      return constructor.newInstance(target, source);
   .............
    }
  }
```

注释1：findBindingConstructorForClass  找到一个通过apt生成的类， 详细代码：

```java
  private static Constructor<? extends Unbinder> findBindingConstructorForClass(Class<?> cls) {
    Constructor<? extends Unbinder> bindingCtor = BINDINGS.get(cls);
    if (bindingCtor != null) {
      if (debug) Log.d(TAG, "HIT: Cached in binding map.");
      return bindingCtor;
    }
    String clsName = cls.getName();
    if (clsName.startsWith("android.") || clsName.startsWith("java.")) {
      if (debug) Log.d(TAG, "MISS: Reached framework class. Abandoning search.");
      return null;
    }
    try {
        // 通过clsName拼接出xxx_ViewBinding，获取到构造器。
      Class<?> bindingClass = cls.getClassLoader().loadClass(clsName + "_ViewBinding");
      //noinspection unchecked
      bindingCtor = (Constructor<? extends Unbinder>) bindingClass.getConstructor(cls, View.class);
      if (debug) Log.d(TAG, "HIT: Loaded binding class and constructor.");
    } catch (ClassNotFoundException e) {
      if (debug) Log.d(TAG, "Not found. Trying superclass " + cls.getSuperclass().getName());
      bindingCtor = findBindingConstructorForClass(cls.getSuperclass());
    } catch (NoSuchMethodException e) {
      throw new RuntimeException("Unable to find binding constructor for " + clsName, e);
    }
    BINDINGS.put(cls, bindingCtor);
    return bindingCtor;
  }
```

注释2：初始化了一个apt生成的实例。最终代码就进入了XXX_ViewBinding 类的构造方法了。



#### XXX_ViewBinding 分析

XXX_ViewBinding 实例代码：

```java
public class ButterKnifeActivity_ViewBinding implements Unbinder {
  private ButterKnifeActivity target;

  private View view2131296350;

  @UiThread
  public ButterKnifeActivity_ViewBinding(ButterKnifeActivity target) {
    this(target, target.getWindow().getDecorView());
  }

  @UiThread
  public ButterKnifeActivity_ViewBinding(final ButterKnifeActivity target, View source) {
    this.target = target;

    View view;
    view = Utils.findRequiredView(source, R.id.tv, "field 'tv' and method 'onViewClicked'");
    target.tv = Utils.castView(view, R.id.tv, "field 'tv'", Button.class);
    view2131296350 = view;
    view.setOnClickListener(new DebouncingOnClickListener() {
      @Override
      public void doClick(View p0) {
        target.onViewClicked(p0);
      }
    });
  }

  @Override
  @CallSuper
  public void unbind() {
    ButterKnifeActivity target = this.target;
    if (target == null) throw new IllegalStateException("Bindings already cleared.");
    this.target = null;
    target.tv = null;
    view2131296350.setOnClickListener(null);
    view2131296350 = null;
  }
}
```

##### 控件赋值

从上面代码可以发现，在构造方法中，通过``Utils.findRequiredView``初始化view实例，最终也是``findViewById``赋值的。

##### 点击事件

控件赋值之后，关键类：``DebouncingOnClickListener``:

```java
public abstract class DebouncingOnClickListener implements View.OnClickListener {
  static boolean enabled = true;

  private static final Runnable ENABLE_AGAIN = new Runnable() {
    @Override public void run() {
      enabled = true;
    }
  };

  @Override public final void onClick(View v) {
    if (enabled) {
      enabled = false;
      v.post(ENABLE_AGAIN);
        //调用回调
      doClick(v);
    }
  }

  public abstract void doClick(View v);
}
```

是一个抽象类，实现View.OnClickListener，onClick方法调用了 doClick的抽象方法。

最终会调用 ``  target.onViewClicked(p0);``onViewClicked方法是含有注解OnClick的方法。最终完成了点击事件的回调。

### Apt的代码生成

其中一个``@BindView``示例:

```java
// butterknife-compiler/src/main/java/butterknife/compiler/ButterKnifeProcessor.java

private void parseBindView(Element element, Map<TypeElement, BindingSet.Builder> builderMap,
      Set<TypeElement> erasedTargetNames) {
    TypeElement enclosingElement = (TypeElement) element.getEnclosingElement();

    // Start by verifying common generated code restrictions.
    boolean hasError = isInaccessibleViaGeneratedCode(BindView.class, "fields", element)
        || isBindingInWrongPackage(BindView.class, element);

    // Verify that the target type extends from View.
    TypeMirror elementType = element.asType();
    if (elementType.getKind() == TypeKind.TYPEVAR) {
      TypeVariable typeVariable = (TypeVariable) elementType;
      elementType = typeVariable.getUpperBound();
    }
    Name qualifiedName = enclosingElement.getQualifiedName();
    Name simpleName = element.getSimpleName();
    if (!isSubtypeOfType(elementType, VIEW_TYPE) && !isInterface(elementType)) {
      if (elementType.getKind() == TypeKind.ERROR) {
        note(element, "@%s field with unresolved type (%s) "
                + "must elsewhere be generated as a View or interface. (%s.%s)",
            BindView.class.getSimpleName(), elementType, qualifiedName, simpleName);
      } else {
        error(element, "@%s fields must extend from View or be an interface. (%s.%s)",
            BindView.class.getSimpleName(), qualifiedName, simpleName);
        hasError = true;
      }
    }

    if (hasError) {
      return;
    }

    // Assemble information on the field.
    int id = element.getAnnotation(BindView.class).value();
    BindingSet.Builder builder = builderMap.get(enclosingElement);
    Id resourceId = elementToId(element, BindView.class, id);
    if (builder != null) {
      String existingBindingName = builder.findExistingBindingName(resourceId);
      if (existingBindingName != null) {
        error(element, "Attempt to use @%s for an already bound ID %d on '%s'. (%s.%s)",
            BindView.class.getSimpleName(), id, existingBindingName,
            enclosingElement.getQualifiedName(), element.getSimpleName());
        return;
      }
    } else {
      builder = getOrCreateBindingBuilder(builderMap, enclosingElement);
    }

    String name = simpleName.toString();
    TypeName type = TypeName.get(elementType);
    boolean required = isFieldRequired(element);

    builder.addField(resourceId, new FieldViewBinding(name, type, required));

    // Add the type-erased version to the valid binding targets set.
    erasedTargetNames.add(enclosingElement);
  }

```

