title: 实现屏幕下方展示的TAB分页
date: 2013-09-24 10:33
categories: android 
---
参考helloandroid兄的腾讯微博应用，我整理了一下
<!--more-->

完整项目在[原文](http://helloandroid.iteye.com/), 首先是效果图：

![](http://dl.iteye.com/upload/attachment/533565/d7ef8d0f-5087-39bb-82da-74ac6374164a.jpg)

梳理知识点总结如下： 

1. TabActivity的使用

```
public class MainActivity extends TabActivity {

	private TabHost tabHost;

	@Override
	public void onCreate(Bundle savedInstanceState) {

		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		tabHost = getTabHost();
		populateTab();

	}

	/**
	 * 组装tab控件
	 */
	private void populateTab() {

		Resources res = getResources();

		populateTabItem(R.drawable.tab_home_selector,
				res.getString(R.string.tab_home), new Intent(this,
						HomeActivity.class));
		populateTabItem(R.drawable.tab_atme_selector,
				res.getString(R.string.tab_refer), new Intent(this,
						ReferActivity.class));
		populateTabItem(R.drawable.tab_message_selector,
				res.getString(R.string.tab_secret), new Intent(this,
						MessageActivity.class));
		populateTabItem(R.drawable.tab_explore_selector,
				res.getString(R.string.tab_search), new Intent(this,
						SearchActivity.class));
		populateTabItem(R.drawable.tab_focus_selector,
				res.getString(R.string.tab_attention), new Intent(this,
						AttentionActivity.class));

	}

	/**
	 * 生成tab_item
	 * 
	 * @param imageResourceSelector
	 *            图片选择器
	 * @param text
	 *            文本
	 * @param intent
	 *            intent
	 */
	private void populateTabItem(int imageResourceSelector, String text,
			Intent intent) {

		View view = View.inflate(this, R.layout.tab_item, null);// 拼装view
		((ImageView) view.findViewById(R.id.tab_item_imageview))
				.setImageResource(imageResourceSelector);
		((TextView) view.findViewById(R.id.tab_item_textview)).setText(text);

		TabSpec spec = tabHost.newTabSpec(text).setIndicator(view)
				.setContent(intent);// 将view注入spec
		tabHost.addTab(spec);

	}

}

```
```
<?xml version="1.0" encoding="UTF-8"?>

<TabHost android:id="@android:id/tabhost" android:layout_width="fill_parent"
	android:layout_height="fill_parent" xmlns:android="http://schemas.android.com/apk/res/android">

	<RelativeLayout android:orientation="vertical"
		android:layout_width="fill_parent" android:layout_height="fill_parent">

		<include android:id="@+id/head_line" layout="@layout/head_line"
			android:layout_width="fill_parent" android:layout_height="wrap_content" />

		<FrameLayout android:id="@android:id/tabcontent"
			android:layout_below="@id/head_line" android:layout_width="fill_parent"
			android:layout_height="fill_parent" android:layout_weight="1.0" />

		<TabWidget android:id="@android:id/tabs" android:background="@drawable/tab_bkg"
			android:layout_gravity="bottom" android:layout_height="60.0dip"
			android:layout_width="fill_parent" android:fadingEdge="none"
			android:fadingEdgeLength="0.0px" android:paddingLeft="0.0dip"
			android:paddingTop="2.0dip" android:paddingRight="0.0dip"
			android:paddingBottom="0.0dip" android:layout_alignParentBottom="true"
			android:layout_alignParentTop="false" />

	</RelativeLayout>

</TabHost>

```
可以看到，TabActivity是继承自Activity，包含了一个TabHost组件。TabHost组件则是继承自FrameLayout的ViewGroup。 

TabHost组件本身的id必须是@android:id/tabhost，它必须包含一个FrameLayout，并且该FrameLayout的id必须是@android:id/tabcontent，此外还要包含一个TabWidget，id是@android:id/tabs。 FrameLayout可以放置每个单独的Activity，而TabWidget则是每个Tab页签。默认第一个页签对应的Activity，会首先显示在FrameLayout里。然后每次点击其他的Tab页签，对应的Activity就会切换显示到FrameLayout里。这个有点类似html中的frameset的概念 

2. 在main.xml中有一行

```
<include android:id="@+id/head_line" layout="@layout/head_line"
			android:layout_width="fill_parent" android:layout_height="wrap_content" />
```
作用是引入另一个View文件，head_line.xml

```
<RelativeLayout android:background="@drawable/header"
	android:layout_width="fill_parent" android:layout_height="wrap_content"
	xmlns:android="http://schemas.android.com/apk/res/android">

	<Button android:id="@+id/top_btn_left" android:textColor="@color/button_text_selector"
		android:background="@drawable/top_refresh_selector"
		android:layout_width="wrap_content" android:layout_height="wrap_content"
		android:layout_marginLeft="12.0dip" android:layout_alignParentLeft="true"
		android:layout_centerVertical="true" />

	<Button android:id="@+id/top_btn_right" android:textColor="@color/button_text_selector"
		android:background="@drawable/top_edit_selector" android:layout_width="wrap_content"
		android:layout_height="wrap_content" android:layout_marginRight="12.0dip"
		android:layout_alignParentRight="true" android:layout_centerVertical="true" />

	<TextView android:id="@+id/top_title" android:textSize="22.0sp"
		android:textColor="@color/head_line_text" android:ellipsize="middle"
		android:gravity="center_horizontal" android:layout_width="wrap_content"
		android:layout_height="wrap_content" android:text="@string/user_name"
		android:singleLine="true" android:layout_toLeftOf="@id/top_btn_right"
		android:layout_toRightOf="@id/top_btn_left"
		android:layout_centerInParent="true"
		android:layout_alignWithParentIfMissing="true" />

</RelativeLayout>
```

用这种方式可以实现View组件的复用，是很方便的，可以学习一下这种方式。把要复用的View写在单独的xml文件里，然后在其他需要的地方，只要直接include就可以了 

3. 每个Tab页签对应的View是tab_item.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<LinearLayout android:orientation="vertical"
	android:layout_width="fill_parent" android:layout_height="wrap_content"
	xmlns:android="http://schemas.android.com/apk/res/android">

	<ImageView android:id="@+id/tab_item_imageview"
		android:layout_width="fill_parent" android:layout_height="32.0dip"
		android:scaleType="fitCenter" />

	<TextView android:id="@+id/tab_item_textview"
		android:layout_width="fill_parent" android:layout_height="wrap_content"
		android:gravity="center" android:singleLine="true"
		android:marqueeRepeatLimit="1" android:textSize="11.0sp"
		android:ellipsize="marquee" />

</LinearLayout>

```
然后在java代码中进行组装

```
View view = View.inflate(this, R.layout.tab_item, null);// 拼装view
		((ImageView) view.findViewById(R.id.tab_item_imageview))
				.setImageResource(imageResourceSelector);
		((TextView) view.findViewById(R.id.tab_item_textview)).setText(text);

		TabSpec spec = tabHost.newTabSpec(text).setIndicator(view)
				.setContent(intent);// 将view注入spec
		tabHost.addTab(spec);

```
这部分的详细说明，可以看google提供的API 

4. 然后这个页面中用到了selector的概念，即当要动态改变某些组件的属性，如颜色，字体大小等，可以用selector来进行动态选择，这里有点类似CSS中的伪类的概念

```
android:textColor="@color/button_text_selector"
```

```
<?xml version="1.0" encoding="UTF-8"?>

<selector xmlns:android="http://schemas.android.com/apk/res/android">

	<item android:state_pressed="true" android:color="@color/button_text_pressed" />

	<item android:state_selected="true" android:color="@color/button_text_pressed" />

	<item android:color="@color/button_text_normal" />

</selector>

```

上面代码的意思是，根据按钮控件是否按下，是否选择，在运行时动态决定颜色。通过同样的方式，还可以动态决定一个按钮的图片

```
<selector
  xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_focused="false" android:state_selected="false" android:state_pressed="false" android:drawable="@drawable/top_edit_normal" />
    <item android:state_focused="false" android:state_selected="true" android:state_pressed="false" android:drawable="@drawable/top_edit_press" />
    <item android:state_focused="true" android:state_selected="false" android:state_pressed="false" android:drawable="@drawable/top_edit_normal" />
    <item android:state_focused="true" android:state_selected="true" android:state_pressed="false" android:drawable="@drawable/top_edit_press" />
    <item android:state_pressed="true" android:drawable="@drawable/top_edit_press" />
</selector>
```

5. 这个页面还用到了一个比较特殊的技巧，就是通过xml，而不是静态图片来绘制背景

```
<RelativeLayout android:background="@drawable/header"
	android:layout_width="fill_parent" android:layout_height="wrap_content"
	xmlns:android="http://schemas.android.com/apk/res/android">

```
上面代码中的android:background="@drawable/header"，指向drawable文件夹中的header.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<shape android:shape="rectangle" xmlns:android="http://schemas.android.com/apk/res/android">

	<gradient android:startColor="#ff6c7078" android:endColor="#ffa6abb5"
		android:angle="270.0" android:type="linear" />

</shape>

```
可以看到，这个控件的背景色，是用xml绘制出来的。不过这个技巧我在别的地方见的比较少，感觉比较冷门