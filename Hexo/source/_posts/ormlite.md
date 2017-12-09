---
title: OrmLite 快速入门
date: 2016-12-14 11:43:17
category: Android 进阶
---
在Android项目开发中，经常会使用到数据库，这个时候，就需要用到OrmLite了，它的英文全称是Object Relational Mapping，意思是对象关系映射；如果接触过Java EE开发的，一定知道Java Web开发就有一个类似的数据库映射框架——Hibernate。简单来说，就是我们定义一个实体类，利用这个框架，它可以帮我们吧这个实体映射到我们的数据库中，在Android中是SQLite，数据中的字段就是我们定义实体的成员变量。
### (一)下载OrmLite Jar
首先去[OrmLite官网下载Jar包](http://ormlite.com/releases/)，Android需要下载两个Jar包:Core 列和 Android 列各下载一个Jar包.
![OrmLite官网截图](/uploads/ormlite.png)
将两个jar包，放到项目中的libs文件夹，并作为项目的Library.

### (二)配置Bean类
每一个Bean类，相当于都对应数据库中的一张表，在OrmLite中，不需要写sql语句，只需要在对应的字段上，添加需要的注解就够了.
```java
@DatabaseTable(tableName = "tb_people")
public class PeopleBean {

    @DatabaseField(generatedId = true)
    private int id;

    @DatabaseField(columnName = "name")
    private String name;

    @DatabaseField(columnName = "age")
    private int age;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
PeopleBean类名上的@DatabaseTable(tableName = “tb_people”)，标明了这个Bean类对象的数据库表名，表名为tb_people。
id属性上的@DatabaseField(generatedId = true) 该属性为数据库主键id,并且自动生成
age、name 属性上的@DatabaseField(columnName = "name”)、@DatabaseField(columnName = "age") 指定了该属性对应的数据库字段名称.
其他注解说明如下
![OrmLite官网截图](/uploads/ormlite_zhujie.png)

### (三)DataBaseHelper类
继承OrmLiteSqliteOpenHelper类，OrmLiteSqliteOpenHelper是SQLiteOpenHelper的子类.
```java
public class DBHelper extends OrmLiteSqliteOpenHelper {
    private static Context mContext;
    private static DBHelper instance;
    public static final int DATABASE_VERSION = 2;   //2016-12-14 11:33:59, 如果数据库表有新增字段,version+1
    public static final String DATA_BASE_NAME = “mytest.db”;	//数据库名称

    private Map<String, Dao> daoMaps = new HashMap<>();

    public DBHelper(Context context) {
        super(context, DATA_BASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase, ConnectionSource connectionSource) {	//数据库第一次创建的时候，执行该方法
        try {
            TableUtils.createTable(connectionSource, PeopleBean.class);	//创建PeopleBean.class对应的数据库表
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, ConnectionSource connectionSource, int i, int i1) {	//当数据库版本升级的时候，会执行该方法，该方法会删除旧数据库表，然后创建新的数据库表，如果PeopleBean有新增字段，但是不升级version,将会报找不到该数据库字段错误
        try {
            TableUtils.dropTable(connectionSource, PeopleBean.class, true);
            TableUtils.createTable(connectionSource, PeopleBean.class);
            onCreate(sqLiteDatabase, connectionSource);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void initOrmLite(Context context) {
        mContext = context;
        getInstance();
    }

    public static DBHelper getInstance() {
        if (instance == null) {
            synInit(mContext);
        }
        return instance;
    }

    private synchronized static void synInit(Context context) {
        if (instance == null) {
            instance = new DBHelper(context);
        }
    }

    /**
     * 获得Dao
     *
     * @param clazz
     * @return
     * @throws SQLException
     */
    public synchronized Dao getDao(Class clazz) throws SQLException {	//每个表都会有一个单独的Dao用于操作
        Dao dao;
        String className = clazz.getSimpleName();

        if (daoMaps.containsKey(className)) {
            dao = daoMaps.get(className);
        } else {
            dao = super.getDao(clazz);
            daoMaps.put(className, dao);
        }
        return dao;
    }

    /**
     * 释放资源
     */
    @Override
    public void close() {
        super.close();
        for (String key : daoMaps.keySet()) {
            Dao dao = daoMaps.get(key);
            dao = null;
        }
    }
}
```
onCreate(SQLiteDatabase sqLiteDatabase, ConnectionSource connectionSource)方法和onUpgrade(SQLiteDatabase database, ConnectionSource connectionSource, int oldVersion, int newVersion)是必须实现的两个方法，onCreate方法会在数据库创建的时候调用，onUpdate会在数据库版本更新的时候调用.
这里将Dao写成单例，对外公布获取该单例的方法。
每个表都会有一个单独的Dao用于操作，通过getDao方法，获得Dao.
```java
public class PeopleDao {
    public static PeopleDao mSearchInstance;
    private Dao<PeopleBean, Integer> mPeopleDao;

    private PeopleDao() {
        try {
            mPeopleDao = DBHelper.getInstance().getDao(PeopleBean.class);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static PeopleDao getInstance() {
        if (mSearchInstance == null) {
            mSearchInstance = new PeopleDao();
        }
        return mSearchInstance;
    }

    /**
     * 插入一条记录
     */
    public void insertPeople(PeopleBean Peoplebean) {
        try {
            mPeopleDao.create(Peoplebean);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 插入多条记录
     */
    public void insertPeople(List<PeopleBean> peoples) {
        try {
            mPeopleDao.create(peoples);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 查询所有记录
     */
    public List<PeopleBean> quearyAllPeople() {
        QueryBuilder<PeopleBean, Integer> queryBuilder = mPeopleDao.queryBuilder();
        try {
            queryBuilder.where();
            PreparedQuery<PeopleBean> preparedQuery = queryBuilder.prepare();
            return mPeopleDao.query(preparedQuery);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

     /**
     * 多条件查询,查询指定名字和年龄的人
     *
     * @param name
     * @param age
     * @return
     */
    public List<PeopleBean> quearyAllHistory(String name, int age) {
        QueryBuilder<PeopleBean, Integer> queryBuilder = mHistoryDao.queryBuilder();
        try {
            queryBuilder.where().eq("name", name).and().eq("age", age);
            PreparedQuery<PeopleBean> preparedQuery = queryBuilder.prepare();
            return mPeopleDao.query(preparedQuery);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 根据id删除记录
     */
    public void deletePeople(int id) {
        try {
            mPeopleDao.deleteById(id);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 更新记录
     */
    public void updatePeople(PeopleBean people) {
        try {
            mPeopleDao.update(people);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
一般的操作就是这些增删改查了.