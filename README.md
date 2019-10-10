### LitePal for Android

LitePal 是一个Android开源库，它提供了很多简洁的API，使得我们可以更容易使用SQLite数据库。
你可以不用写一句SQL语句就可以完成大部分数据库操作，包括创建表，更新表，约束操作，聚合功能等等。


### Features

- 使用对象关系映射(ORM) 模型。
- 几乎零配置(只有一个配置文件，该配置文件属性很少)。
- 自动维护所有表格(比如创建、更改、删除表格)。
- 提供封装的API，无需写SQL语句。
- 很棒的集群查询功能。
- 依然可以选择使用SQL，LitePal提供比原始更易用更好的API接口。


### 优点

- 配置简单，只有一个配置文件；
- API简洁，调用方便；
- 结构清晰，但依赖包却很小；
- 丰富的API，除了CRUD操作，还提供了多库操作和CRUD的异步操作；
- 关于开源库的使用，作者提供了系列博文来进行了详细说明，使得集成非常方便。

### 缺点

- 使用反射影响性能；
- 整个框架很关键的一点是id自增长，这支撑着整个框架的运转，但却使得我们很不方便维护已有的id属性，需要通过一定的转换，方能正常存储数据。


### 资料

[https://github.com/LitePalFramework/LitePal](https://github.com/LitePalFramework/LitePal)

[郭霖的原创博客-LitePal系列文章](https://blog.csdn.net/sinyu890807/article/category/2522725)

### 集成步骤

#### 1. 添加依赖
```
dependencies {
    implementation 'org.litepal.android:core:2.0.0'
}
```
#### 2. 创建配置文件：litepal.xml

- 配置文件存放位置：项目的assets文件夹
- 配置文件内容：
```
<?xml version="1.0" encoding="utf-8"?>
<litepal>

    <dbname value="litepal_demo" />
    <version value="1" />
    <list>
        <mapping class = "com.cys.litepaldemo.bean.News"></mapping>
    </list>

</litepal>
```
- dbname 标签用于配置工程的数据库文件名
- version 用于配置数据库的版本信息，注意：每次升级数据库，该版本号加1。
- list 用于配置映射类。注意：mapping里面的路径一定要是全路径名称

#### 3. 初始化：配置LitePalApplication

> 进行数据库操作需要用到Context，所以需要在初始化的时候将Context传到LitePal

- 如果没有自定义Application，则可以直接使用LitePalApplication，在AndroidManifest.xml中配置
```
<manifest>
    <application
        android:name="org.litepal.LitePalApplication"
        ...
    >
    ...
    </application>
</manifest>
```
- 如果有自定义的Application，则可以直接继承LitePalApplication类
- 或者在自定义的Application类里，onCreate方法里调用LitePal.initialize(this);

#### 4. 创建表格

> LitePal采取的是对象关系映射(ORM)的模式，简单点说，我们使用的编程语言是面向对象语言，而我们使用的数据库则是关系型数据库，那么将面向对象的语言和面向关系的数据库之间建立一种映射关系，这就是对象关系映射了。基于对象关系映射模式，根据对象关系映射模式的理念，每一张表都应该对应一个模型(Model)。

- 首先定义model，例如定义一个model：News
```
public class News {
    
    private int id;

    private String title;

    private String content;

    private Date publishDate;

    private int commentCount;
    
    /* getter and setter */
}
```

- 在litepal.xml配置文件中添加映射
```
    <mapping class = "com.cys.litepaldemo.model.News"></mapping>
```
- 在下一次操作数据库的时候，数据库表格会自动生成。

#### 5. 升级表格

> LitePal中升级表格的方法很简单，我们不需要编写任何与升级相关的逻辑，也不需要关心程序是从哪个版本升级过来的，唯一要做的就是确定好最新的Model结构是什么样的，然后将litepal.xml中的版本号加1，所有的升级逻辑就都会自动完成了。

- 新增、删除表格：创建或删除相应的bean，修改配置文件litepal.xml中的映射表，同时将version属性加1
- 新增、删除字段：修改响应的bean，同时将配置文件litepal.xml中的version属性加1
- 在下次进行数据库操作时，表格会自动升级

#### 6. 建立表关联

> LitePal 可以自动建立表关联，我们不需要关心什么外键、中间表等实现的细节，只需要在对象中声明好它们相互之间的引用关系，LitePal就会自动在数据库表之间建立好相应的关联关系

- 一对一关联：表示两个表中的数据必须是一一对应的关系


    例如我们想在新闻模型中新增一个导语和摘要，导语和摘要放在同一张表里，一篇新闻对应一个导语和摘要，两者是一一对应的关系。
    那么我们可以新建一个Introduction模型，同时在配置文件里更新映射表
    
```
public class Introduction {
    private int id;

    private String guide;

    private String digest;
}
```
    
    然后在News里增加Introduction属性：

```
public class News {
    
    ...
    
    private Introduction introduction;
    
    /* getter and setter */
}
```

    这样News表格和Introduction表格会自动建立关联

---    

- 一对多关联：表示一张表中的数据可以对应另一张表中的多条数据


    例如我们想在新闻模型中新增一个评论列表，一篇新闻可以对应多条评论记录，而一条评论对应一篇文章
    那么我们可以新建一个Comment模型，同时在配置文件里更新映射表
    
```
public class Comment {

    private int id;

    private String content;

    private News news;
}
```
    
    然后在News里增加commentList属性：

```
public class News {
    
    ...
    
    private List<Comment> commentList = new ArrayList<Comment>();
    
    /* getter and setter */
}
```

    这样News表格和Comment表格会自动建立关联
    
- 多对多关联：表示两张关联表中的数据都可以对应另一张表中的多条数据。


    例如我们的频道列表，一个频道列表对应多条新闻，而一条新闻同时也可能对应多个频道
    我们也只需要新建对应的Channel模型，然后在模型中指定对应的其他模型

```
public class Channel {

    private int id;

    private String name;

    private List<News> newsList = new ArrayList<News>();
}
```
    然后在News里增加channelList属性：

```
public class News {
    
    ...
    
    private List<Channel> channelList = new ArrayList<Channel>();
    
    /* getter and setter */
}
```

    这样News表格和Comment表格会自动建立关联
    
    
#### 7. 数据库操作

> 前面介绍了数据库表格的建表、更新表、关联表的操作，那么如何进行数据的增删改查呢？LitePal中提供了LitePalSupport类，并且要求所有实体类都要继承自LitePalSupport，LitePalSupport中提供了增删改查操作的方法，实体类就拥有进行增删改查的能力了

```
public class News extends LitePalSupport {
    ...
}
```

1. 增加数据：

方法名 | 适用范围 | 返回值
---|---|---
save() | 存储一条数据到表中，返回存储结果 | boolean
saveThrows() | 存储一条数据到表中，失败会抛出异常 | 无
saveOrUpdate() | 不存在时则新增，存在时则更新 | boolean
saveIfNotExist() | 不存在时新增记录 | boolean
saveAsync() | 异步存储 | SaveExecutor
LitePal.saveAll() | 将集合中的数据都存储到表中 | 无
LitePal.saveAllAsync() | 异步将集合中的数据都存储到表中 | SaveExecutor
```
News news = new News();
news.setTitle("这是一条新闻标题");
news.setContent("这是一条新闻内容");
news.setPublishDate(new Date());
news.save();

news.saveAsync().listen(new SaveCallback() {
    @Override
    public void onFinish(boolean success) {
        
    }
});

```
2. 删除数据

方法名 | 适用范围 | 返回值
---|---|---
delete() | 删除一条数据到表中，返回删除结果 | boolean
deleteAsync() | 异步删除 | UpdateOrDeleteExecutor

```
news.delete();

news.deleteAsync().listen(new UpdateOrDeleteCallback() {
    @Override
    public void onFinish(int rowsAffected) {
        
    }
});
```

3. 修改数据

方法名 | 适用范围 | 返回值
---|---|---
update() | 更新一条数据到表中，返回更新结果 | boolean
updateAsync() | 异步更新 | UpdateOrDeleteExecutor

```
news.update();

news.updateAsync(news.getId()).listen(new UpdateOrDeleteCallback() {
    @Override
    public void onFinish(int rowsAffected) {

    }
});
```

4. 查询数据：

- 普通查询


    - modelClass 是对应的model,id是数据的id值,isEager标志是否要同时查询关联表的数据
    
    - DataSupport.find(Class<T> modelClass, long id, boolean isEager)
    - DataSupport.findAsync(final Class<T> modelClass, final long id, final boolean isEager)
    - DataSupport.findFirst(Class<T> modelClass, boolean isEager)
    - DataSupport.findFirstAsync(final Class<T> modelClass, final boolean isEager)
    - DataSupport.findLast(Class<T> modelClass, boolean isEager)
    - DataSupport.findLastAsync(final Class<T> modelClass, final boolean isEager)
    - DataSupport.findAll(Class<T> modelClass, boolean isEager, long... ids)
    - DataSupport.findAllAsync(final Class<T> modelClass,final boolean isEager,final long... ids)
    - DataSupport.findBySQL(String... sql)
    
- 连缀查询

    
    - DataSupport.select(String... conditions)
    - DataSupport.where(String... conditions)
    - DataSupport.order(String... conditions)
    - DataSupport.limit(int value)
    - DataSupport.offset(int value)
    
举例：
```
List<News> newsList = DataSupport
        .select("title", "content")
		.where("commentcount > ?", "0")
		.order("publishdate desc")
		.limit(10)
		.offset(10)
		.find(News.class);
```

#### 8. 聚合函数

> LitePal中一共提供了count()、sum()、average()、max()和min()这五种聚合函数

```
int result = DataSupport.where("commentcount = ?", "0").count(News.class);
int result = DataSupport.sum(News.class, "commentcount", int.class);
double result = DataSupport.average(News.class, "commentcount");
int result = DataSupport.max(News.class, "commentcount", int.class);
int result = DataSupport.min(News.class, "commentcount", int.class);
```

### 源码分析

1. 初始化方法：将Context传给LitePal
```
    /**
     * Initialize to make LitePal ready to work. If you didn't configure LitePalApplication
     * in the AndroidManifest.xml, make sure you call this method as soon as possible. In
     * Application's onCreate() method will be fine.
     *
     * @param context
     * 		Application context.
     */
    public static void initialize(Context context) {
        LitePalApplication.sContext = context;
    }
```

2. 获取数据库对象：每次操作数据库前都需要获取数据库对象，所以在第一次使用的时候进行解析配置文件、建表、更新表等操作

    
```
    // LitePal.java
    /**
     * Get a writable SQLiteDatabase.
     *
     * @return A writable SQLiteDatabase instance
     */
    public static SQLiteDatabase getDatabase() {
        synchronized (LitePalSupport.class) {
            return Connector.getDatabase();
        }
    }


    // Connector.java
    /**
	 * Call getDatabase directly will invoke the getWritableDatabase method by
	 * default.
	 * 
	 * This is method is alias of getWritableDatabase.
	 * 
	 * @return A writable SQLiteDatabase instance
	 */
	public static SQLiteDatabase getDatabase() {
		return getWritableDatabase();
	}
	
	/**
	 * Get a writable SQLiteDatabase.
	 * 
	 * There're a lot of ways to operate database in android. But LitePal
	 * doesn't support using ContentProvider currently. The best way to use
	 * LitePal well is get the SQLiteDatabase instance and use the methods like
	 * SQLiteDatabase#save, SQLiteDatabase#update, SQLiteDatabase#delete,
	 * SQLiteDatabase#query in the SQLiteDatabase class to do the database
	 * operation. It will be improved in the future.
	 * 
	 * @return A writable SQLiteDatabase instance
	 */
	public synchronized static SQLiteDatabase getWritableDatabase() {
		LitePalOpenHelper litePalHelper = buildConnection();
		return litePalHelper.getWritableDatabase();
	}
```

3. 解析配置文件litepal.xml：LitePalOpenHelper.buildConnection()
```
    // LitePalOpenHelper.java
    /**
	 * Build a connection to the database. This progress will analysis the
	 * litepal.xml file, and will check if the fields in LitePalAttr are valid,
	 * and it will open a SQLiteOpenHelper to decide to create tables or update
	 * tables or doing nothing depends on the version attributes.
	 * 
	 * After all the stuffs above are finished. This method will return a
	 * LitePalHelper object.Notes this method could throw a lot of exceptions.
	 * 
	 * @return LitePalHelper object.
	 * 
	 * @throws org.litepal.exceptions.InvalidAttributesException
	 */
	private static LitePalOpenHelper buildConnection() {
		LitePalAttr litePalAttr = LitePalAttr.getInstance();
		litePalAttr.checkSelfValid();
		if (mLitePalHelper == null) {
			String dbName = litePalAttr.getDbName();
			if ("external".equalsIgnoreCase(litePalAttr.getStorage())) {
				dbName = LitePalApplication.getContext().getExternalFilesDir("") + "/databases/" + dbName;
			} else if (!"internal".equalsIgnoreCase(litePalAttr.getStorage()) && !TextUtils.isEmpty(litePalAttr.getStorage())) {
                // internal or empty means internal storage, neither or them means sdcard storage
                String dbPath = Environment.getExternalStorageDirectory().getPath() + "/" + litePalAttr.getStorage();
                dbPath = dbPath.replace("//", "/");
                if (BaseUtility.isClassAndMethodExist("android.support.v4.content.ContextCompat", "checkSelfPermission") &&
                        ContextCompat.checkSelfPermission(LitePalApplication.getContext(), Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                    throw new DatabaseGenerateException(String.format(DatabaseGenerateException.EXTERNAL_STORAGE_PERMISSION_DENIED, dbPath));
                }
                File path = new File(dbPath);
                if (!path.exists()) {
                    path.mkdirs();
                }
                dbName = dbPath + "/" + dbName;
            }
			mLitePalHelper = new LitePalOpenHelper(dbName, litePalAttr.getVersion());
		}
		return mLitePalHelper;
	}
	
	// LitePalAttr.java
	/**
	 * Provide a way to get the instance of LitePalAttr.
	 * @return the singleton instance of LitePalAttr
	 */
	public static LitePalAttr getInstance() {
		if (litePalAttr == null) {
			synchronized (LitePalAttr.class) {
				if (litePalAttr == null) {
					litePalAttr = new LitePalAttr();
                    loadLitePalXMLConfiguration();
				}
			}
		}
		return litePalAttr;
	}

	private static void loadLitePalXMLConfiguration() {
        if (BaseUtility.isLitePalXMLExists()) {
            LitePalConfig config = LitePalParser.parseLitePalConfiguration();
            litePalAttr.setDbName(config.getDbName());
            litePalAttr.setVersion(config.getVersion());
            litePalAttr.setClassNames(config.getClassNames());
            litePalAttr.setCases(config.getCases());
            litePalAttr.setStorage(config.getStorage());
        }
    }
```

4. LitePalOpenHelper：继承自SQLiteOpenHelper，如果数据库文件不存在会调用onCreate，存在则调用onUpgrade
```
    @Override
	public void onCreate(SQLiteDatabase db) {
		Generator.create(db);
	}

	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
		Generator.upgrade(db);
		SharedUtil.updateVersion(LitePalAttr.getInstance().getExtraKeyName(), newVersion);
	}
```


5. Generator：
```
    /**
	 * Create tables based on the class models defined in the litepal.xml file.
	 * After the tables are created, add association to these tables based on
	 * the associations between class models.
	 * 
	 * @param db
	 *            Instance of SQLiteDatabase.
	 */
	static void create(SQLiteDatabase db) {
		create(db, true);
		addAssociation(db, true);
	}
	
	/**
	 * Upgrade tables to make sure when model classes are changed, the
	 * corresponding tables in the database should be always synchronized with
	 * them.
	 * 
	 * @param db
	 *            Instance of SQLiteDatabase.
	 */
	static void upgrade(SQLiteDatabase db) {
		drop(db);
		create(db, false);
		updateAssociations(db);
		upgradeTables(db);
		addAssociation(db, false);
	}
```

6. 
