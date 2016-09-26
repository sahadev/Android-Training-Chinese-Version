原文地址：[http://android.xsoftlab.net/training/basics/data-storage/databases.html](http://android.xsoftlab.net/training/basics/data-storage/databases.html)

对于保存重复的结构化的数据最理想的方式就是存到数据库，比如联系人信息。这节课假定你有SQL数据库基础，会帮助你开始学习Android上的SQLite数据库。你将通过使用[android.database.sqlite](http://android.xsoftlab.net/reference/android/database/sqlite/package-summary.html)包下API来操作数据库。

##定义架构以及契约
SQL数据库的主要原则之一就是架构：一份数据库组织结构的正式声明。架构反映在SQL语句中，你可以使用它来创建数据库。你可能会发现它对于创建一个辅助类来说是非常有帮助的。我们所已知的contract类，它明确的指明了在系统化和自我记录化的方式下的布局模式。

contract类是一个契约容器，它定义了URI、TABLE、COLUMN的名称。contract类允许你使用相同的契约访问处于同一个包下的所有类。它可以使你在一个地方改了列名，然后让这个更改消息在代码中传播(**Tips：**这里应该说的是观察者模式)。

组织一个contract类的最好方式就是在类的根级中定义全局的数据库。然后对每一个表都创建一个内部类并且在内部类中列举出它的每一列列名。

> **Note:**内部类可以通过实现[BaseColumns](http://android.xsoftlab.net/reference/android/provider/BaseColumns.html)接口，继承一个被称为_ID的主键，一些Android中的类比如cursor adapter会期望拥有这个主键，不过这不是必须的，但是它可以有助于数据库在Android框架层中运行的更协调。

下面的小片段定义了一个单表，并列举出了它表名和列名：
```java
public final class FeedReaderContract {
    // To prevent someone from accidentally instantiating the contract class,
    // give it an empty constructor.
    public FeedReaderContract() {}
    /* Inner class that defines the table contents */
    public static abstract class FeedEntry implements BaseColumns {
        public static final String TABLE_NAME = "entry";
        public static final String COLUMN_NAME_ENTRY_ID = "entryid";
        public static final String COLUMN_NAME_TITLE = "title";
        public static final String COLUMN_NAME_SUBTITLE = "subtitle";
        ...
    }
}
```

##使用SQL Helper创建一个数据库
一旦定义了数据库的模型，你应该实现可以创建并可以为维护数据库以及表的方法。下面是创建表、删除表的一些典型声明：
```java
private static final String TEXT_TYPE = " TEXT";
private static final String COMMA_SEP = ",";
private static final String SQL_CREATE_ENTRIES =
    "CREATE TABLE " + FeedEntry.TABLE_NAME + " (" +
    FeedEntry._ID + " INTEGER PRIMARY KEY," +
    FeedEntry.COLUMN_NAME_ENTRY_ID + TEXT_TYPE + COMMA_SEP +
    FeedEntry.COLUMN_NAME_TITLE + TEXT_TYPE + COMMA_SEP +
    ... // Any other options for the CREATE command
    " )";
private static final String SQL_DELETE_ENTRIES =
    "DROP TABLE IF EXISTS " + FeedEntry.TABLE_NAME;
```

就像保存在内部存储器上的文件，Android会将你的数据库文件保存到与应用程序有关的私有磁盘空间内。所以你的数据是安全的，因为默认情况下这块区域是不可以被其它应用程序访问的。

在[SQLiteOpenHelper](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteOpenHelper.html)类中有一系列非常有用的API。当使用这个类的API操作数据库的引用时，系统会潜在的执行耗时操作。所以创建和更新数据库的时间，应该放在需要操作的时候操作，而不是在应用启动的时候操作。你需要操作的所有事情需要通过调用[getWritableDatabase()](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteOpenHelper.html#getWritableDatabase())或者[getReadableDatabase()](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteOpenHelper.html#getReadableDatabase())来完成。

> **Note:**因为这些操作可能会长时间运行，所以要确保调用getWritableDatabase()或getReadableDatabase()的线程是工作线程，比如使用[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)或者[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)。

为了使用[SQLiteOpenHelper](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteOpenHelper.html)，需要创建SQLiteOpenHelper的一个子类然后重写onCreate(), onUpgrade()和onOpen()回调方法。你可能还需要实现onDowngrade()，但在这里不是必须的。
下面是SQLiteOpenHelper的实现样例，其中使用了上面所示的部分代码：
```java
public class FeedReaderDbHelper extends SQLiteOpenHelper {
    // If you change the database schema, you must increment the database version.
    public static final int DATABASE_VERSION = 1;
    public static final String DATABASE_NAME = "FeedReader.db";
    public FeedReaderDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(SQL_CREATE_ENTRIES);
    }
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // This database is only a cache for online data, so its upgrade policy is
        // to simply to discard the data and start over
        db.execSQL(SQL_DELETE_ENTRIES);
        onCreate(db);
    }
    public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        onUpgrade(db, oldVersion, newVersion);
    }
}
```

如果需要访问数据库，需要实例化SQLiteOpenHelper的子类：
```java
FeedReaderDbHelper mDbHelper = new FeedReaderDbHelper(getContext());
```

##放置信息到数据库
通过往[insert()](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteDatabase.html#insert(java.lang.String,%20java.lang.String,%20android.content.ContentValues))方法中传递[ContentValues](http://android.xsoftlab.net/reference/android/content/ContentValues.html)对象可以向数据库中插入数据：
```java
// Gets the data repository in write mode
SQLiteDatabase db = mDbHelper.getWritableDatabase();
// Create a new map of values, where column names are the keys
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_ENTRY_ID, id);
values.put(FeedEntry.COLUMN_NAME_TITLE, title);
values.put(FeedEntry.COLUMN_NAME_CONTENT, content);
// Insert the new row, returning the primary key value of the new row
long newRowId;
newRowId = db.insert(
         FeedEntry.TABLE_NAME,
         FeedEntry.COLUMN_NAME_NULLABLE,
         values);
```

insert()方法的第一个参数是一个简单的表名，第二个参数提供了当ContentValues为空的时候framework插入NULL值的那一列的列名(如果你想设置这个参数为"null"，那么当插入的值为空的时候framework将不会空值插入到数据库)。

##从数据库中读取信息
为了可以从数据库读取数据，应该使用[query()](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteDatabase.html#query(boolean,%20java.lang.String,%20java.lang.String[],%20java.lang.String,%20java.lang.String[],%20java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String))方法，传入你的选择条件以及期望返回的那几列。这个方法结合了insert()方法和update()方法的基础，定义了你想匹配的那几列，但并不插入数据。查询返回的结果会在[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)对象中。
```java
SQLiteDatabase db = mDbHelper.getReadableDatabase();
// Define a projection that specifies which columns from the database
// you will actually use after this query.
String[] projection = {
    FeedEntry._ID,
    FeedEntry.COLUMN_NAME_TITLE,
    FeedEntry.COLUMN_NAME_UPDATED,
    ...
    };
// How you want the results sorted in the resulting Cursor
String sortOrder =
    FeedEntry.COLUMN_NAME_UPDATED + " DESC";
Cursor c = db.query(
    FeedEntry.TABLE_NAME,  // The table to query
    projection,                               // The columns to return
    selection,                                // The columns for the WHERE clause
    selectionArgs,                            // The values for the WHERE clause
    null,                                     // don't group the rows
    null,                                     // don't filter by row groups
    sortOrder                                 // The sort order
    );
```

为了查看cursor中的每一行，需要使用Cursor的移动方法，在你开始读取数据之前你需要一直调用这个方法。通常情况下，你应该在最开始调用[moveToFirst()](http://android.xsoftlab.net/reference/android/database/Cursor.html#moveToFirst())方法，它会将读取位置放置到第一个结果实体所在的位置。对于每一行来说，你可以通过调用Cursor类中的其中一个方法读取一个列值，比如使用[getString()](http://android.xsoftlab.net/reference/android/database/Cursor.html#getString(int)),[getLong()](http://android.xsoftlab.net/reference/android/database/Cursor.html#getLong(int))。对于每一个get方法，你必须给这些方法传递一个要读取那一列的索引位置值，你可以通过[getColumnIndex()](http://android.xsoftlab.net/reference/android/database/Cursor.html#getColumnIndex(java.lang.String))或者[getColumnIndexOrThrow()](http://android.xsoftlab.net/reference/android/database/Cursor.html#getColumnIndexOrThrow(java.lang.String))获得索引位置值：
```java
cursor.moveToFirst();
long itemId = cursor.getLong(
    cursor.getColumnIndexOrThrow(FeedEntry._ID)
);
```

##从数据库中删除信息
如果要从数据库表中删除一行，你需要提供可以标识那一行的选择条件。数据库API提供了一种机制，用于创建选择条件，这种机制可以防止被SQL注入。这种机制会将选择规范划分为选择条件和选择参数。选择条件用于定义查找的列，也允许同时查找多列。选择参数的值会测试绑定到条件上的对应值。因为结果不会像常规的SQL语句那样被处理，它会用于免疫SQL注入的。
```java
// Define 'where' part of query.
String selection = FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
// Specify arguments in placeholder order.
String[] selectionArgs = { String.valueOf(rowId) };
// Issue SQL statement.
db.delete(table_name, selection, selectionArgs);
```

##更新数据库
如果需要修改数据库值的子集，请使用[update()](http://android.xsoftlab.net/reference/android/database/sqlite/SQLiteDatabase.html#update(java.lang.String,%20android.content.ContentValues,%20java.lang.String,%20java.lang.String[]))方法。

更新表的方法结合了insert()的内容语法与delete()的where语法。
```java
SQLiteDatabase db = mDbHelper.getReadableDatabase();
// New value for one column
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_TITLE, title);
// Which row to update, based on the ID
String selection = FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
String[] selectionArgs = { String.valueOf(rowId) };
int count = db.update(
    FeedReaderDbHelper.FeedEntry.TABLE_NAME,
    values,
    selection,
    selectionArgs);
```

