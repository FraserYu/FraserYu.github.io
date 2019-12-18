---
title: EasyExcel读取Excel实际应用
tags:
  - EasyExcel
  - POI
  - Spring Boot
  - Excel
categories: [Coding, Java]
id: easyexcel-read
date: 2019-10-07 18:32:34
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
description: 通过EasyExcel读写Excel，极大的增加了业务灵活性，同时降低了内存溢出的风险
keywords: EasyExcel,Excel,读取Excel,写入Excel
icons: [fas fa-fire accent]
---



## 写在前面
Java 后端程序员应该会遇到读取 Excel 信息到 DB 等相关需求，脑海中可能突然间想起 Apache POI 这个技术解决方案，但是当 Excel 的数据量非常大的时候，你也许发现，POI 是将整个 Excel 的内容全部读出来放入到内存中，所以内存消耗非常严重，如果同时进行包含大数据量的 Excel 读操作，很容易造成内存溢出问题

但 EasyExcel 的出现很好的解决了 POI 相关问题，原本一个 3M 的 Excel 用 POI 需要100M左右内存, 而 EasyExcel 可以将其降低到几 M，同时再大的 Excel 都不会出现内存溢出的情况，因为是逐行读取 Excel 的内容 (老规矩，这里不用过分关心下图，脑海中有个印象即可，看完下面的用例再回看这个图，就很简单了)

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-17-09-51.jpg)


另外 EasyExcel 在上层做了模型转换的封装，不需要 cell 等相关操作，让使用者更加简单和方便，且看

## 简单读
假设我们 excel 中有以下内容:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-09-09-50.png)


我们需要新建 User 实体，同时为其添加成员变量
```java
@Data
public class User {

	/**
	 * 姓名
	 */
	@ExcelProperty(index = 0)
	private String name;

	/**
	 * 年龄
	 */
	@ExcelProperty(index = 1)
	private Integer age;
}
```

你也许关注到了 `@ExcelProperty` 注解，同时使用了 index 属性 (0 代表第一列，以此类推)，该注解同时支持以「列名」name 的方式匹配，比如:
```java
@ExcelProperty("姓名")
private String name;
```

按照 github 文档的说明:
>  不建议 index 和 name 同时用，要么一个对象只用index，要么一个对象只用name去匹配

1. 如果读取的 Excel 模板信息列固定，这里建议以 index 的形式使用，因为如果用名字去匹配，名字重复，会导致只有一个字段读取到数据，所以 index 是更稳妥的方式
2. 如果 Excel 模板的列 index 经常有变化，那还是选择 name 方式比较好，不用经常性修改实体的注解 index 数值

所以大家可以根据自己的情况自行选择

编写测试用例
```java
@Test
public void readExcel(){
    String fileName = TestUtils.getPath() + "excel" + File.separator + "users1.xlsx";
    EasyExcel.read(fileName, User.class, new UserExcelListener()).sheet().doRead();
}
```
EasyExcel 类中重载了很多个 read 方法，这里不一一列举说明，请大家自行查看；同时 sheet 方法也可以指定 sheetNo，默认是第一个 sheet 的信息

上面代码的 `new UserExcelListener()` 异常醒目，这也是 EasyExcel 逐行读取 Excel 内容的关键所在，自定义 `UserExcelListener` 继承 `AnalysisEventListener`

```java
@Slf4j
public class UserExcelListener extends AnalysisEventListener<User> {

	/**
	 * 批处理阈值
	 */
	private static final int BATCH_COUNT = 2;
	List<User> list = new ArrayList<User>(BATCH_COUNT);

	@Override
	public void invoke(User user, AnalysisContext analysisContext) {
		log.info("解析到一条数据:{}", JSON.toJSONString(user));
		list.add(user);
		if (list.size() >= BATCH_COUNT) {
			saveData();
			list.clear();
		}
	}

	@Override
	public void doAfterAllAnalysed(AnalysisContext analysisContext) {
		saveData();
		log.info("所有数据解析完成！");
	}

	private void saveData(){
		log.info("{}条数据，开始存储数据库！", list.size());
		log.info("存储数据库成功！");
	}
}
```

到这里请回看文章开头的 EasyExcel 原理图，invoke  方法逐行读取数据，对应的就是订阅者 1；doAfterAllAnalysed 方法对应的就是订阅者 2，这样你理解了吗？

打印结果:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-15-56-41.png)


从这里可以看出，虽然是逐行解析数据，但我们可以自定义阈值，完成数据的批处理操作，可见 EasyExcel 操作的灵活性

## 自定义转换器
这是最基本的数据读写，我们的业务数据通常不可能这么简单，有时甚至需要将其转换为程序可读的数据
### 性别信息转换
比如 Excel 中新增「性别」列，其性别为男/女，我们需要将 Excel 中的性别信息转换成程序信息: 「1: 男；2:女」

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-16-04-49.png)


首先在 User 实体中添加成员变量 gender:
```java
@ExcelProperty(index = 2)
private Integer gender;
```

EasyExcel 支持我们自定义 converter，将 excel 的内容转换为我们程序需要的信息，这里新建 GenderConverter，用来转换性别信息
```java
public class GenderConverter implements Converter<Integer> {

	public static final String MALE = "男";
	public static final String FEMALE = "女";

	@Override
	public Class supportJavaTypeKey() {
		return Integer.class;
	}

	@Override
	public CellDataTypeEnum supportExcelTypeKey() {
		return CellDataTypeEnum.STRING;
	}

	@Override
	public Integer convertToJavaData(CellData cellData, ExcelContentProperty excelContentProperty, GlobalConfiguration globalConfiguration) throws Exception {
		String stringValue = cellData.getStringValue();
		if (MALE.equals(stringValue)){
			return 1;
		}else {
			return 2;
		}
	}

	@Override
	public CellData convertToExcelData(Integer integer, ExcelContentProperty excelContentProperty, GlobalConfiguration globalConfiguration) throws Exception {
		return null;
	}
}
```

>  上面程序的 Converter 接口的泛型是指要转换的 Java 数据类型，与 supportJavaTypeKey 方法中的返回值类型一致

打开注解 `@ExcelProperty` 查看，该注解是支持自定义 Converter 的，所以我们为 User 实体添加 **gender** 成员变量，并指定 converter
```java
/**
 * 性别 1：男；2：女
 */
@ExcelProperty(index = 2, converter = GenderConverter.class)
private Integer gender;
```

来看运行结果:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-16-09-14.png)


数据按照我们预期做出了转换，从这里也可以看出，Converter 可以一次定义到处是用的便利性

### 日期信息转换
日期信息也是我们常见的转换数据，比如 Excel 中新增「出生年月」列，我们要解析成 **yyyy-MM-dd** 格式，我们需要将其进行格式化，EasyExcel 通过 `@DateTimeFormat` 注解进行格式化

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-16-11-16.png)


在 User 实体中添加成员变量 **birth**，同时应用 `@DateTimeFormat` 注解，按照要求做格式化
```java
/**
 * 出生日期
 */
@ExcelProperty(index = 3)
@DateTimeFormat("yyyy-MM-dd HH:mm:ss")
private String birth;
```

来看运行结果:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-16-13-08.png)


如果这里你指定 birth 的类型为 Date，试试看，你得到的结果是什么？

到这里都是以测试的方式来编写程序代码，作为 Java Web 开发人员，尤其在目前主流 Spring Boot 的架构下，所以如何实现 Web 方式读取 Excel 的信息呢？
## web 读
### 简单 Web
很简单，只是将测试用例的关键代码移动到 Controller 中即可，我们新建一个 `UserController`，在其添加 `upload` 方法
```java
@RestController
@RequestMapping("/users")
@Slf4j
public class UserController {
	@PostMapping("/upload")
	public String upload(MultipartFile file) throws IOException {
		EasyExcel.read(file.getInputStream(), User.class, new UserExcelListener()).sheet().doRead();
		return "success";
	}
}
```

其实在写测试用例的时候你也许已经发现，listener 是以 new 的形式作为参数传入到 EasyExcel.read 方法中的，这是不符合 Spring IoC 的规则的，我们通常读取 Excel 数据之后都要针对读取的数据编写一些业务逻辑的，而业务逻辑通常又会写在 Service 层中，我们如何在 listener 中调用到我们的 service 代码呢？

**先不要向下看，你脑海中有哪些方案呢？ **

### 匿名内部类方式
匿名内部类是最简单的方式，我们需要先新建 Service 层的信息:
新建 IUser 接口:
```java
public interface IUser {
	public boolean saveData(List<User> users);
}
```
新建 IUser 接口实现类 UserServiceImpl:
```java
@Service
@Slf4j
public class UserServiceImpl implements IUser {
	@Override
	public boolean saveData(List<User> users) {
		log.info("UserService {}条数据，开始存储数据库！", users.size());
		log.info(JSON.toJSONString(users));
		log.info("UserService 存储数据库成功！");
		return true;
	}
}
```

接下来，在 Controller 中注入 IUser:
```java
@Autowired
private IUser iUser;
```

修改 upload 方法，以匿名内部类重写 listener 方法的形式来实现:
```java
@PostMapping("/uploadWithAnonyInnerClass")
	public String uploadWithAnonyInnerClass(MultipartFile file) throws IOException {
		EasyExcel.read(file.getInputStream(), User.class, new AnalysisEventListener<User>(){
			/**
			 * 批处理阈值
			 */
			private static final int BATCH_COUNT = 2;
			List<User> list = new ArrayList<User>();

			@Override
			public void invoke(User user, AnalysisContext analysisContext) {
				log.info("解析到一条数据:{}", JSON.toJSONString(user));
				list.add(user);
				if (list.size() >= BATCH_COUNT) {
					saveData();
					list.clear();
				}
			}

			@Override
			public void doAfterAllAnalysed(AnalysisContext analysisContext) {
				saveData();
				log.info("所有数据解析完成！");
			}

			private void saveData(){
				iUser.saveData(list);
			}
		}).sheet().doRead();
		return "success";
	}
```
查看结果:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-06/2019-10-07-17-57-32.png)


这种实现方式，其实这只是将 listener 中的内容全部重写，并在 controller 中展现出来，当你看着这么臃肿的 controller 是不是非常难受？很显然这种方式不是我们的最佳编码实现

### 构造器传参
在之前分析 SpringBoot 统一返回源码时，不知道你是否发现，Spring 底层源码多数以构造器的形式传参，所以我们可以将为 listener 添加有参构造器，将 Controller 中依赖注入的 IUser 以构造器的形式传入到 listener :

```java
@Slf4j
public class UserExcelListener extends AnalysisEventListener<User> {

	private IUser iUser;

	public UserExcelListener(IUser iUser){
		this.iUser = iUser;
	}

    // 省略相应代码...

    private void saveData(){
		iUser.saveData(list); //调用 userService 中的 saveData 方法
	}
	
```

更改 Controller 方法:
```java
@PostMapping("/uploadWithConstructor")
public String uploadWithConstructor(MultipartFile file) throws IOException {
    EasyExcel.read(file.getInputStream(), User.class, new UserExcelListener(iUser)).sheet().doRead();
    return "success";
}
```

运行结果: 同上

这样更改后，controller 代码看着很清晰，但如果后续业务还有别的 Service 需要注入，我们难道要一直添加有参构造器吗？很明显，这种方式同样不是很灵活。

其实在使用匿名内部类的时候，你也许会想到，我们可以通过 Java8 lambda 的方式来解决这个问题

### Lambda 传参
为了解决构造器传参的痛点，同时我们又希望 listener 更具有通用性，没必要为每个 Excel 业务都新建一个 listener，因为 listener 都是逐行读取 Excel 数据，只需要将我们的业务逻辑代码传入给 listener 即可，所以我们需用到 `Consumer<T>` ，将其作为构造 listener 的参数。

新建一个工具类 ExcelDemoUtils，用来构造 listener:
```java
public class ExcelDemoUtils {

	/**
	 * 指定阈值
	 * @param consumer
	 * @param threshold
	 * @param <T>
	 * @return
	 */
	public static <T> AnalysisEventListener<T> getListener(Consumer<List<T>> consumer, int threshold) {
		return new AnalysisEventListener<T>() {
			private LinkedList<T> linkedList = new LinkedList<T>();

			@Override
			public void invoke(T t, AnalysisContext analysisContext) {
				linkedList.add(t);
				if (linkedList.size() == threshold){
					consumer.accept(linkedList);
					linkedList.clear();
				}
			}

			@Override
			public void doAfterAllAnalysed(AnalysisContext analysisContext) {
				if (linkedList.size() > 0){
					consumer.accept(linkedList);
				}
			}
		};
	}

	/**
	 * 不指定阈值，阈值默认为10
	 * @param consumer
	 * @param <T>
	 * @return
	 */
	public static <T> AnalysisEventListener<T> getListener(Consumer<List<T>> consumer){
		return getListener(consumer, 10);
	}
```
我们看到，getListener 方法接收一个 `Consumer<List<T>>` 的参数，这样下面代码被调用时，我们的业务逻辑也就会被相应的执行了:
```java
consumer.accept(linkedList);
```

继续改造 Controller 方法:
```java
@PostMapping("/uploadWithLambda")
public String uploadWithLambda(MultipartFile file) throws IOException {
    AnalysisEventListener<User> userAnalysisEventListener = ExcelDemoUtils.getListener(this.batchInsert(), 2);
    EasyExcel.read(file.getInputStream(), User.class, userAnalysisEventListener).sheet().doRead();
    return "success";
}


private Consumer<List<User>> batchInsert(){
    //其他业务逻辑只需要添加到该方法中即可
    return users -> iUser.saveData(users);
}
```

运行结果: 同上

到这里，我们只需要将业务逻辑定制在 `batchInsert` 方法中:
1. 满足 Controller RESTful API 的简洁性
2. listener 更加通用和灵活，它更多是扮演了抽象类的角色，具体的逻辑交给抽象方法的实现来完成
3. 业务逻辑可扩展性也更好，逻辑更加清晰

## 总结
到这里，关于如何使用 EasyExcel 读取 Excel 信息的基本使用方式已经介绍完了，还有很多细节内容没有讲，大家可以自行查阅 [EasyExcel Github](https://github.com/alibaba/easyexcel) 文档去发现更多内容

除了读取 Excel 的读取，还有 Excel 的写入，如果需要将其写入到指定位置，配合 HuTool 的工具类 FileWriter 的使用是非常方便的，针对 EasyExcel 的使用，如果大家有什么问题，也欢迎到博客下方探讨

完整代码请在公众号回复「demo」，点开链接，查看「easyexceldemo」文件夹的内容即可



## 感谢
非常感谢 EasyExcel 的作者 🌹🌹，让 Excel 的读写更加方便 

## 灵魂追问
1. 除了 Consumer，如果需要返回值的业务逻辑，需要用到哪个函数式接口呢？
2. 当出现复杂表头的时候要如何处理呢？
3. 将 DB 数据写入到 Excel 并下载，如何实现呢？
4. 从 EasyExcel 的设计上，你学到了什么，欢迎博客下方留言讨论

## 提高效率工具

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png) 


--------

## 推荐阅读
1. [这次走进并发的世界，请不要错过](https://dayarch.top/2019/08/25/bing-fa-bian-cheng-zhi-chu-tan/)
2. [学并发编程，透彻理解这三个核心是关键](https://dayarch.top/2019/08/28/bing-fa-bian-cheng-san-da-he-xin/)
3. [并发Bug之源有三，请睁大眼睛看清它们](https://dayarch.top/2019/09/04/bing-fa-bian-cheng-san-da-wen-ti/)
4. [可见性有序性，Happens-before来搞定](https://dayarch.top/2019/09/12/you-xu-xing-ke-jian-xing-happens-before-lai-gao-ding/)
5. [解决原子性问题？你首先需要的是宏观理解](https://dayarch.top/2019/09/19/jie-jue-yuan-zi-xing-wen-ti-ni-shou-xian-xu-yao-de-shi-hong-guan-li-jie/)
6. [面试并发volatile关键字时，我们应该具备哪些谈资？](https://dayarch.top/2019/09/29/mian-shi-bing-fa-volatile-guan-jian-zi-shi-wo-men-ying-gai-ju-bei-na-xie-tan-zi/)


--------

> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享 
> - 高效工具汇总 | 回复「工具」
> - 面试问题分析与解答 
> - 技术资料领取 | 回复「资料」

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)