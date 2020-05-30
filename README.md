# middle-test




功能1:时间戳的显示

1.为了实现时间戳的显示,先在layout.plus_time.xml(原noteslist_item.xml)中修改视图


   <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/textView"
            android:layout_width="208dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:text="TextView" />

        <TextView
            android:id="@+id/textView3"
            android:layout_width="209dp"
            android:layout_height="57dp"
            android:layout_weight="2"
            android:text="TextView3" />
    </LinearLayout>
    
    
2.在NotePadProvider中修改创建时间和修改时间的类型为TEXT


   @Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " TEXT"
                   + ");"

           );
        //   db.execSQL("drop table "+NotePad.Notes.TABLE_NAME);

        //   onUpgrade(db,1,2);
       }
       

3.将修改日期借由SimpleCursorAdapter放入list中


   String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

        int[] viewIDs = { R.id.textView ,R.id.textView3 };

        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,                             // The Context for the ListView
                      R.layout.plus_time,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs
              );

        setListAdapter(adapter);
        

4.创建修改日期格式的方法,用于将长整型的数据转换为时间数据
 
 public static String getFormatedDateTime(String pattern, long dateTime) {
        SimpleDateFormat sDateFormat = new SimpleDateFormat(pattern);
        return sDateFormat.format(new Date(dateTime + 0));
    }
    

5.在NotePadProvider中的insert方法里修改创建数据时时间的格式

        String now=getFormatedDateTime("yyyy-MM-dd HH:mm:ss",System.currentTimeMillis());
            
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
        }
        
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
        }




6.依法炮制在NoteEditor的更新方法中改变日期的显示格式
 
 private final void updateNote(String text, String title) {

        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,getFormatedDateTime("yyyy-MM-dd HH:mm:ss",System.currentTimeMillis()));

     ....
     ....
     }





2.增加搜索日记的功能
大概思路:在标题栏添加一个搜索按钮,点击按钮后进入一个新界面进行搜索


1.在list_options_menu中加入搜索按钮

   <item
        android:id="@+id/search"

        android:icon="@android:drawable/ic_menu_zoom"
        android:title="Search"
        app:actionViewClass="android.widget.SearchView"
        app:showAsAction="always" />
        
        
        
2.添加新类NoteSearch作为搜索的新界面
该类继承ListActivity并实现SearchView.OnQueryTextListener接口

##########################################################################################################
 ##########################################################################################################
 ##########################################################################################################
 
 public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {

    private SearchView mSearchView;
    private SimpleCursorAdapter mAdapter;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
    };@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        mSearchView = (SearchView)findViewById(R.id.search_view);

        mSearchView.setIconifiedByDefault(false);
        mSearchView.setSubmitButtonEnabled(true); //显示搜索按钮

        mSearchView.setOnQueryTextListener(this);  //注册监听器
    }

    @Override
    public boolean onQueryTextChange(String s) { //Test改变的时候执行的内容
        //Text发生改变时执行的内容
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+s+"%" };//查询条件参数，配合selection参数使用,%通配多个字符

        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。
        //managedQuery(Uri, String[], String, String[], String)等同于Context.getContentResolver().query()
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.用于ContentProvider查询的URI，从这个URI获取数据
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date.用于标识uri中有哪些columns需要包含在返回的Cursor对象中
                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择
                selectionArgs,                    // 查询条件参数，配合selection参数使用
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.查询结果的排序方式，按照某个columns来排序，例：String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE
        );

        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { R.id.textView , R.id.textView3 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                   //context:上下文
                R.layout.plus_time,         //layout:布局文件，至少有int[]的所有视图
                cursor,                          //cursor：游标
                dataColumns,                     //from：绑定到视图的数据
                viewIDs                          //to:用来展示from数组中数据的视图
                //flags：用来确定适配器行为的标志，Android3.0之后淘汰
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    public boolean onQueryTextSubmit(String s) {
        return false;
    }
}



###########################################################################################################
##########################################################################################################
##########################################################################################################
 
2.5  在mainfests中加入NoteSearch的intent


 <activity
            android:name=".NoteSearch"
            android:label="NoteSearch"
            >

            <intent-filter>
                <action android:name="android.intent.action.NoteSearch" />
                <action android:name="android.intent.action.SEARCH" />
                <action android:name="android.intent.action.SEARCH_LONG_PRESS" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
                <!--1.vnd.android.cursor.dir代表返回结果为多列数据-->
                <!--2.vnd.android.cursor.item 代表返回结果为单列数据-->
            </intent-filter>
        </activity>


3.NoteSearch使用note_search视图
 
 <?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="请输入搜索内容..."
        >
    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
 

4.在NotesList的onOptionsItemSelected中添加监听
 
   public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        case R.id.menu_add:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_INSERT. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
           startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
           return true;
        case R.id.menu_paste:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_PASTE. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
          startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
          return true;

          case R.id.search:

                startActivity(new Intent(Intent.ACTION_SEARCH,getIntent().getData()));
                return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
 
 
