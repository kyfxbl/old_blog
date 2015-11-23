title: 基于hibernate4的分页组件和自定义标签
date: 2013-09-24 11:33
categories: java
---
本文介绍一种分页组件的完整代码，最后封装了一个简单的jsp自定义标签 
<!--more-->

# 页面效果 

![](http://dl.iteye.com/upload/attachment/0073/0954/43f90c6b-c21b-306b-842a-ecf3ee134f8a.png)

没加CSS，只是意思一下。这个分页的功能比较简单，不过更复杂的分页功能，原理也是差不多的 

# Page对象

```
public class Page {

	private static final int DEFAULT_PAGE_SIZE = 10;

	private static final int DEFAULT_CURRENT_PAGE = 1;

	private int currentPage;// 当前页数，通常在Action层设置

	private int pageSize;// 每页记录数，通常在Action层设置

	private int totalCount;// 总记录数，在DAO层设置

	public Page(int currentPage, int pageSize) {
		this.currentPage = currentPage;
		this.pageSize = pageSize;
	}

	public Page(int currentPage) {
		this.currentPage = currentPage;
		this.pageSize = DEFAULT_PAGE_SIZE;
	}

	public Page() {
		this.currentPage = DEFAULT_CURRENT_PAGE;
		this.pageSize = DEFAULT_PAGE_SIZE;
	}

	public int getFirstIndex() {
		return pageSize * (currentPage - 1);
	}

	public boolean hasPrevious() {
		return currentPage > 1;
	}

	public boolean hasNext() {
		return currentPage < getTotalPage();
	}

	public int getTotalPage() {

		long remainder = totalCount % this.getPageSize();

		if (0 == remainder) {
			return totalCount / this.getPageSize();
		}

		return totalCount / this.getPageSize() + 1;
	}
}
```

省略了必须的getter和setter方法。这里主要有3个字段currentPage、pageSize、totalCount，分别表示当前页、每页记录数、总记录数。基本上所有的分页组件，都需要这些字段 

其实总页数totalPage也是必须的，但是这个字段是由pageSize和totalCount算出来的，所以即时计算得到比较好，如果也作为一个字段的话，那么和另外2个字段就不正交 

getFirstIndex()方法也很重要，因为后面查询数据库的时候，需要作为第一条记录的标识，作为参数传给查询数据库的方法 

# DAO层写法

```
public interface IBookDAO extends IGenericDAO<Book> {

	public Book queryByIsbn(String isbn);

	public List<Book> queryByNameWithPage(String name, Page page);

	public List<Book> queryByNameWithoutPage(String name);

}
```

其中涉及到分页查询的就是queryByNameWithPage()方法，这个Page对象是从Action传递下来的

```
@Repository
public class BookDAO extends GenericDAO<Book> implements IBookDAO {

	public BookDAO() {
		super(Book.class);
	}

	@Override
	public Book queryByIsbn(String isbn) {
		String hql = "from Book b where b.isbn = ?";
		return queryForObject(hql, new Object[] { isbn });
	}

	@Override
	public List<Book> queryByNameWithPage(String name, Page page) {
		String hql = "from Book b where b.name = ?";
		return queryForList(hql, new Object[] { name }, page);
	}

	@Override
	public List<Book> queryByNameWithoutPage(String name) {
		String hql = "from Book b where b.name = ?";
		return queryForList(hql, new Object[] { name });
	}

}
```

这里调用的是GenericDAO里的方法queryForList()

```
@SuppressWarnings("unchecked")
	protected List<T> queryForList(String hql, Object[] params, Page page) {

		generatePageTotalCount(hql, params, page);

		Query query = sessionFactory.getCurrentSession().createQuery(hql);
		setQueryParams(query, params);
		query.setFirstResult(page.getFirstIndex());
		query.setMaxResults(page.getPageSize());
		return query.list();
	}

	/**
	 * 该方法会改变参数page的totalCount字段
	 * 
	 * @param originHql
	 *            原始hql语句
	 * @param params
	 *            原始参数
	 * @param page
	 *            页面对象
	 */
	private void generatePageTotalCount(String originHql, Object[] params,
			Page page) {
		String generatedCountHql = "select count(*) " + originHql;
		Query countQuery = sessionFactory.getCurrentSession().createQuery(
				generatedCountHql);
		setQueryParams(countQuery, params);
		int totalCount = ((Long) countQuery.uniqueResult()).intValue();
		page.setTotalCount(totalCount);
	}

```
这段代码关键有2点。一个是调用Page对象的getFirstIndex()方法和getPageSize()方法，确定查询结果的范围。另一个是设置Page对象自身的totalCount字段，因为Action里持有该Page对象的引用，所以根据totalCount和pageSize计算出totalPage之后，就可以在前台显示出总页数 

# Action层的写法

```
@Controller
@Scope("prototype")
public class BookAction extends ActionSupport {

	private static final long serialVersionUID = -4400311497910666205L;

	@Autowired
	private IBookService bookService;

	private Page page;// 分页组件

	private List<Book> books;// 查询结果列表

	public String list() {
		if (null == page) {
			page = new Page();// 如果page对象为空，说明不是通过点击页码跳转
		}
		books = bookService.getBooks(page);
		return SUCCESS;
	}
}
```

这里省略了与分页无关的方法和字段，以及getter、setter方法 

这里重点是list()方法，如果是通过jsp页面点击“上一页”、“下一页”的链接进入此方法的，那么struts2框架就会自动初始化Page对象。如果这个时候page对象为空，那么说明是用户直接进入该页面，就需要初始化一个Page对象（currentPage=1，表示是第一页） 

该Page对象此时的totalCount是0，在查询之后，才会根据查询到的记录总数，设置totalCount的值，由于Action持有此Page的引用，所以可以在前台显示出来 

# 前台页面的写法

```
<div id="page_area">

				<s:if test="page.hasPrevious()">
					<a href="list.action?page.currentPage=${page.currentPage-1}">上一页</a>
				</s:if>

				<span>第${page.currentPage}页</span>
				<span>共${page.getTotalPage()}页</span>

				<s:if test="page.hasNext()">
					<a href="list.action?page.currentPage=${page.currentPage+1}">下一页</a>
				</s:if>

			</div>
```

里面用到了struts2的if标签，还有${}表达式。我感觉${}表达式还是很方便的，struts2封装的各种标签我倒是不喜欢用，自己用html标签开发，我觉得更灵活一点 

这个分页组件在各个页面都可以用到，但是如果每个jsp都要拷贝这段代码，就不好了，所以需要把它封装成标签。封装后的效果是这样的：

```
<wfm:page page="${page}" />
```

# 自定义标签

首先需要在WEB-INF目录下创建一个.tld文件，只要放在WEB-INF目录下就可以了，所以一般会单独建一个子目录tld来保存 

![](http://dl.iteye.com/upload/attachment/0073/0972/9e78bce6-3861-38f7-8e8d-7396b804769f.png)

这个tld内容如下
```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
    version="2.0">

    <description>Workforce Tag Lib</description>
    <tlib-version>1.0</tlib-version>
    <short-name>WorkforceTagLibrary</short-name>
    <uri>/wfm-tags</uri>

    <tag>
        <description>generate page bar</description>
        <name>page</name>
        <tag-class>com.huawei.inoc.framework.tag.PageTag</tag-class>
        <body-content>empty</body-content>
        <attribute>
            <name>page</name>
            <required>true</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
    </tag>

</taglib>
```

上面比较重要的元素是<uri>，后面在jsp页面里使用这个标签的时候会用到，然后可以定义多个<tag>元素，每个都是一个标签，命名都很清晰，就不用解释了。其中<rtexprvalue>这个标签，设置为true之后，则此属性支持变量 

下面是标签的实现类
```
public class PageTag extends SimpleTagSupport {

	private Page page;

	@Override
	public void doTag() throws JspException, IOException {
		String content = generateContent();
		getJspContext().getOut().write(content);
	}

	private String generateContent() {

		StringBuilder response = new StringBuilder();

		response.append("<div id=\"page_area\">");

		if (page.hasPrevious()) {
			response.append("<a href=\"list.action?page.currentPage="
					+ (page.getCurrentPage() - 1) + "\">上一页</a>");
		}

		response.append("<span>第" + page.getCurrentPage() + "页</span>");
		response.append("<span>共" + page.getTotalPage() + "页</span>");

		if (page.hasNext()) {
			response.append("<a href=\"list.action?page.currentPage="
					+ (page.getCurrentPage() + 1) + "\">下一页</a>");
		}

		response.append("</div>");

		return response.toString();
	}

	public Page getPage() {
		return page;
	}

	public void setPage(Page page) {
		this.page = page;
	}

}
```

上面的代码很简单，jsp2.0之后，要实现自定义标签是很容易的，继承SimpleTagSupport类，实现doTag()方法即可，如果需要设置一些属性的话，声明为字段，并添加getter和setter方法 

最后是jsp页面的引用方法

```
<%@ taglib prefix="wfm" uri="/wfm-tags"%>
```
```
<wfm:page page="${page}" />
```

在比较早的版本里，还需要在web.xml里增加tag-lib的配置，从某个版本的servlet规范之后（好像是3.0），就不需要了，容器会自动到WEB-INF目录下加载所有的.tld文件