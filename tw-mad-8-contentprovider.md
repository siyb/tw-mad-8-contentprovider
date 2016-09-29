% MAD - Android 8: ContentProvider
% Patrick Sturm
% 29.09.2016

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-8-contentprovider](https://github.com/siyb/tw-mad-8-contentprovider)

# Agenda

## Agenda

* Introduction
* Implementation

# ![Components](./component_contentprovider.png)

# Introduction


## Introduction - 1 - Resources

* Lessons:
    * [http://developer.android.com/guide/topics/providers/content-providers.html](http://developer.android.com/guide/topics/providers/content-providers.html)
    * [http://developer.android.com/guide/topics/providers/content-provider-basics.html](http://developer.android.com/guide/topics/providers/content-provider-basics.html)
    * [http://developer.android.com/guide/topics/providers/content-provider-creating.html](http://developer.android.com/guide/topics/providers/content-provider-creating.html)
* Javadoc: [http://developer.android.com/reference/android/content/ContentProvide](http://developer.android.com/reference/android/content/ContentProvide)

## Introduction - 2 - Android and Databases

* Android uses SQLite
* SQLite is a file based database, meaning that we do not require a DBMS (in the sense of a running database server)
* SQLite started as an extension for TCL (Tool Command Language)
* It supports very few data types: TEXT, NUMERIC, INTEGER, REAL, NONE
  * The data affinity is aweful: [https://www.sqlite.org/datatype3.html](https://www.sqlite.org/datatype3.html)

## Introduction - 3 - ContentProvider Concepts

* ContentProvider are another Android component
    * They need to be defined in the manifest
    * They do have a lifecycle
* ContentProvider are used to access data
    * Usually, the data resides in a database (SQLite)
    * ContentProvider can be adapted to dish out data from different sources
        * XML, JSON (even csv brr)
        * Media (Images, Videos ...)
        * Etc.
* ContentProvider use URIs to point to data
    * You can define a URI format for your data (if you write a ContentProvider)
    * ContentProvider URIs start with content://
    * Somewhat like REST
* You can have a SQLite database without having to result to ContentProviders

## Introduction - 4 - Shortcomings

* ContentProvider do not like joins
    * They are supported, but it's a pain in the *** to implement them
* Check out ContractContact if you want to see for yourself
    * You have to pass the join with the table parameter of query(...)
* Provider are public by default, meaning that everybody can use them, you need to restrict access explicitly
* ContentProvider can conflict if the authority is the same, therefore you have to change the authority for each ContentProvider you use in other projects
    * attachInfo can be used to obtain the authority from the manifest
* Android does not provide an OR mapper -> use proxy objects, unless you use a dedicated framework for it


# Implementation

## Implementation - 1 - Example: Access Content

```java
public MyActivity extends Activity { 
  public void onCreate(Bundle b) {
    // Activity extends Context, remember? 
    Cursor c = getContentResolver()
      .query(MyContentProvider.URI, 
        null, null, null, null); 
    // Can only be called form UI thread, 
    // will spawn a small note on user's screen 
    Toast.makeText(this, 
      String.format(getString(R.string.itemcount), 
    c.getCount()), Toast.LENGTH_LONG).show();
  } 
}
```

## Implementation - 2 - Example: Access Content Explained

* In this case, we are querying a content resolver for data. The query will always return a Cursor
* Cursors are "pointers" to your data

## Implementation - 3 - Example: ContentProvider - Creation

```java
public class MyContentProvider 
  extends ContentProvider { 
  private static final 
    String DATABASE_NAME = "mydatabase.db"; 
  private static final int DATABASE_VERSION = 1; 
  private static final String CONTENTPROVIDER_AUTHORITY = 
    "com.sphericalelephant.MyContentProvider"; 
  public static final Uri CONTENT_URI = 
    Uri.parse("content://" + CONTENTPROVIDER_AUTHORITY); 
  private MyDatabaseHelper databaseHelper; 
  @Override 
  public boolean onCreate() { 
    databaseHelper = new MyDatabaseHelper(getContext()); 
    return true; 
  }
```

## Implementation - 4 - Example: ContentProvider - query

```java
  @Override 
  public Cursor query(Uri uri, String[] projection, 
    String selection, String[] selectionArgs, 
    String sortOrder) { 
    SQLiteDatabase db = databaseHelper
      .getReadableDatabase(); 
    // no groupBy / having support, 
    // DO NOT CLOSE DATABASE -> Cursor becomes useless 
    return db.query(MyDatabaseHelper.TABLE, projection, 
    selection, selectionArgs, null, null, sortOrder); 
  }
```


## Implementation - 5 - Example: ContentProvider - insert

```java
  @Override 
  public Uri insert(Uri uri, ContentValues values) { 
    SQLiteDatabase db = 
      databaseHelper.getWritableDatabase(); 
    // the id of the row, if you 
    // specify a primary key, this will be it 
    // if not you get the row sqlite defines internally 
    int id = db.insertWithOnConflict(
      MyDatabaseAdapter.TABLE, 
      MyDatabaseAdapter.ID, values, 
      SQLiteDatabase.CONFLICT_REPLACE); 
    ...
```

## Implementation - 6 - Example: ContentProvider - insert cont.

```java
    // this URI should be usable by query (ideally) 
    // we are not doing this here 
    Uri newUri = Uri.withAppendedPath(
      CONTENT_URI, "/ID/" + id); 
    // we notify observers so that 
    // they can reload the data if they wish 
    getContext().getContentResolver()
      .notifyChange(newUri, null); 
    // no cursor here, close the db 
    db.close(); 
    return newUri; 
} 
```

## Implementation - 7 - Example: ContentProvider - delete / update

```java
  @Override 
  public int delete(Uri uri, String selection, String[] selectionArgs) { 
    // similar to insert, you will figure it out ;) 
  }
  @Override 
  public int update(Uri uri, ContentValues values, 
    String selection, String[] selectionArgs) { 
    // see delete(...) 
  } 
} 
```

## Implementation - 4 - Manifest

*Since ContentProvider are components (in the Android sense), they need to be defined in the Manifest file (don't mind the wrong use of xml comments pls)

```xml
<provider 
  <!-- The path to our class . 
    denotes our application package --> 
  android:name=
    ".contentprovider.MyContentProvider" 
  <!-- The URI we use to access 
    the contentprovider from code --> 
  android:authorities=
    "com.sphericalelephant
      .contentprovider.mycontentprovider" > 
</provider>
```

# Any Questions?