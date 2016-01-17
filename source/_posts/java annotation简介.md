title: java annotation简介
date: 2013-09-24 11:13
categories: java 
---
开发中自定义annotation的场景不太多，但是很多框架的源码里都用到了自定义annotation，不了解的话就看不懂了，所以也简单地学习一下 
<!--more-->

试了一下，比较简单，以下通过一个例子来说明 

首先是annotation本身的定义

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ValueBind {

	public FieldType type();

	public String value();

}
```

这里用了一个枚举

```
public enum FieldType {

	STRING, INT

}
```

下面是用到了该标签的类

```
public class Student {

	private String id;

	private String name;

	private int age;

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		sb.append("id: " + this.getId() + "\n");
		sb.append("name: " + this.getName() + "\n");
		sb.append("age: " + this.getAge() + "\n");
		return sb.toString();
	}

	public String getId() {
		return id;
	}

	@ValueBind(type = FieldType.STRING, value = "123")
	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	@ValueBind(type = FieldType.STRING, value = "kyfxbl")
	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	@ValueBind(type = FieldType.INT, value = "29")
	public void setAge(int age) {
		this.age = age;
	}
}
```

最后是执行的代码，可以看到效果

```
public class Main {

	public static void main(String[] args) throws Exception {

		Student student = new Student();

		System.out.println(student.toString());

		Method[] methodArray = student.getClass().getDeclaredMethods();

		for (Method method : methodArray) {

			if (method.isAnnotationPresent(ValueBind.class)) {

				ValueBind annotation = method.getAnnotation(ValueBind.class);

				FieldType type = annotation.type();
				String value = annotation.value();

				if (FieldType.INT.equals(type)) {
					method.invoke(student, new Integer(value));
				} else {
					method.invoke(student, value);
				}
			}

		}

		System.out.println(student.toString());
	}
}
```

![](http://dl.iteye.com/upload/attachment/0075/6799/b5fb699b-9a90-37af-bc55-580be65996f3.png) 

我总的感觉，注解本身干不了啥，只能起到一个类型标识的作用（正如其命名annotation一样），然后代码里可以根据注解的类型，再进行需要的处理，相当于annotation起到的是类型标记的作用 

不过在annotation里可以定义一些字段，这样可以传递参数过去 

另外，注解也是可以“继承”的，比如说，在spring框架中，@Controller、@Service、@Repository，都认为是@Component

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any
	 */
	String value() default "";

}
```