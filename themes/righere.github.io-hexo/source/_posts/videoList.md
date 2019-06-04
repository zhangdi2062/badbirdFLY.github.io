---
title: 创建一个简单的视频播放目录
date: 2016-10-08 11:24:28
tags: [android,java]
---

之前一直想自己做一个视频播放器，首先第一步就是做一个视频列表的界面，这个简单的界面目的是将手机内存中的视频收集在一起，点击相应的视频就开始执行播放等操作。创建一个简单的视频列表主要基于一个listview，把所有的视频都放进listview当中，listview的每一个item对应一个视频，每一个视频item显示相应视频的信息，包括视频的名字、大小、时间长度等等，以下主要分5步完成列表目录：
<!-- more -->
# 简单的文件列表布局
创建一个videolistview.xml线性布局
```
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ListView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:dividerHeight="2dp"
            android:id="@+id/video_list">
        </ListView>
    </LinearLayout>
```

每一个videoitem.xml的布局：

这里使用了androidstudio2.2新增的布局ConstraintLayout

  ![](http://of6x0sb2r.bkt.clouddn.com/videoItem.png)

使用了一个imageview用显示视频的略缩图，两个textview分别显示视频的名字和视频长度，上图直接用图形化界面直接创建的，所以代码可能有点长
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:layout_width="150dp"
        android:id="@+id/item_image"
        android:contentDescription="@string/video_thumbnail"
        app:layout_constraintLeft_toLeftOf="parent"
        tools:layout_editor_absoluteY="0dp"
        android:layout_height="150dp"
        tools:ignore="MissingConstraints" />

    <TextView
        android:layout_width="250dp"
        android:id="@+id/video_name"
        android:text="videoname"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_marginTop="5dp"
        android:layout_height="51dp"
        android:layout_marginStart="8dp"
        app:layout_constraintLeft_toRightOf="@+id/item_image"
        android:textAlignment="center"
        android:textStyle="normal|bold"
        app:layout_constraintBottom_toBottomOf="@+id/video_duration"
        app:layout_constraintHorizontal_bias="0.9" />

    <TextView
        android:layout_width="250dp"
        android:id="@+id/video_duration"
        android:text="videodration"
        android:layout_height="50dp"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_marginStart="8dp"
        app:layout_constraintLeft_toRightOf="@+id/item_image"
        android:layout_marginBottom="5dp"
        android:textAlignment="center"
        android:textStyle="normal|bold" />
    
</android.support.constraint.ConstraintLayout>
```

# 创建视频列表中的每一个视频的videoitem类
> 每一个视频列表的videoitem由两部分组成，一是Video的缩略图，二是video的名字和时长等属性,
先创建缩略图的LoadImage.java类，再创建VideoItem.java类

LoadImage.java
```
public class LoadImage {
    Bitmap bitmap;

    public LoadImage(Bitmap bitmap) {
        this.bitmap = bitmap;
    }

    public Bitmap getBitmap() {
        return bitmap;
    }
}
```

VideoItem.java
```
public class VideoItem implements Serializable {

    /**
     * videoItem的属性
     */
    private int id;
    private long mVideoDuration;
    private long mVideoSize;
    private String album;
    private String artist;
    private String displayname;
    private String mVideoTitle;
    private String mVideoMimeType;
    private String path;
    private LoadImage loadImage;

    /**
     * @param id 视频文件的id
     * @param mVideoDuration 视频的文件市场
     * @param mVideoSize 视频文件的大小
     * @param album 文件的专辑
     * @param artist 文件的作者
     * @param displayname 显示文件名
     * @param mVideoTitle 视频目录中的文件头
     * @param mVideoMimeType 文件类型
     * @param path 文件路径
     */
    public VideoItem(int id, long mVideoDuration, long mVideoSize,
                     String album, String artist, String displayname,String mVideoTitle, String mVideoMimeType, String path) {
        super();
        this.id = id;
        this.mVideoDuration = mVideoDuration;
        this.mVideoSize = mVideoSize;
        this.album = album;
        this.artist = artist;
        this.displayname = displayname;
        this.mVideoTitle = mVideoTitle;
        this.mVideoMimeType = mVideoMimeType;
        this.path = path;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public long getmVideoDuration() {
        return mVideoDuration;
    }

    public long getmVideoSize() {
        return mVideoSize;
    }

    public void setmVideoSize(long mVideoSize) {
        this.mVideoSize = mVideoSize;
    }

    public String getmVideoTitle() {
        return mVideoTitle;
    }

    public void setmVideoTitle(String mVideoTitle) {
        this.mVideoTitle = mVideoTitle;
    }

    public String getmVideoMimeType() {
        return mVideoMimeType;
    }

    public void setmVideoMimeType(String mVideoMimeType) {
        this.mVideoMimeType = mVideoMimeType;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }

    public String getAlbum() {
        return album;
    }

    public void setAlbum(String album) {
        this.album = album;
    }

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }

    public String getDisplayname() {
        return displayname;
    }

    public void setDisplayname(String displayname) {
        this.displayname = displayname;
    }

    public LoadImage getLoadImage() {
        return loadImage;
    }

    public void setLoadImage(LoadImage loadImage) {
        this.loadImage = loadImage;
    }

}
```
# 创建一个VideoList的适配器

>重点在适配器中添加缩略图方法addthumbnail，和getview方法

```
public class VideoListAdapter extends BaseAdapter{

    public List<VideoItem> videoItems;
    private LayoutInflater mLayoutInflater;
    private ArrayList<LoadImage> thumbnail = new ArrayList<LoadImage>();

    public VideoListAdapter(Context context, List<VideoItem> videoItems){
        mLayoutInflater = LayoutInflater.from(context);
        this.videoItems = videoItems;
    }

    @Override
    public int getCount() {
        return thumbnail.size();
    }

    public void addthumbnail(LoadImage loadimage){
            thumbnail.add(loadimage);
    }

    @Override
    public Object getItem(int position) {
        return position;
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        if (convertView == null){
            holder = new ViewHolder();
            convertView = mLayoutInflater.inflate(R.layout.videolistitem,null);
            holder.img = (ImageView) convertView.findViewById(R.id.item_image);
            holder.videoname = (TextView) convertView.findViewById(R.id.video_name);
            holder.time = (TextView) convertView.findViewById(R.id.video_duration);
            convertView.setTag(holder);
        }
        else {
            holder = (ViewHolder) convertView.getTag();
        }
            holder.videoname.setText(videoItems.get(position).getDisplayname());
            long min = videoItems.get(position).getmVideoDuration()/1000/60;
            long sec = videoItems.get(position).getmVideoDuration()/1000%60;
            holder.time.setText(min + " : " + sec);
            holder.img.setImageBitmap(thumbnail.get(position).getBitmap());
        return convertView;
    }

    public final class ViewHolder {
        public ImageView img;
        public TextView videoname;
        public TextView time;
    }
}
```

# 获取内存中的视频
>需要创建以上两步只是配置好了布局，后面我们需要从内存中获取视频的文件，首先我们获取到视频文件然后放入List

先实现获取视频list的接口：

AbstractProvider.java
```
public interface AbstractProvider {
    List<VideoItem> getList();
}
```

再实现获取所有视频方法

VideoProvider.java
```
public class VideoProvider implements AbstractProvider {

    private Context context;

    public VideoProvider(Context context) {
        this.context = context;
    }

    @Override
    public List<VideoItem> getList() {
        List<VideoItem> list = null;
        if(context != null){
            Cursor cursor = context.getContentResolver().query(
                    MediaStore.Video.Media.EXTERNAL_CONTENT_URI,null,null,null, MediaStore.Video.Media.DEFAULT_SORT_ORDER);
            Log.i(TAG, "getList: " + MediaStore.Video.Media.EXTERNAL_CONTENT_URI.toString());
            if(cursor != null){
                 list = new ArrayList<VideoItem>();
                while (cursor.moveToNext()){
                    int id = cursor.getInt(cursor.getColumnIndexOrThrow(MediaStore.Video.Media._ID));
                    String title = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.TITLE));
                    String album = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.ALBUM));
                    String artist = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.ARTIST));
                    String displayName = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DISPLAY_NAME));
                    String mime_type = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.MIME_TYPE));
                    String path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DATA));
                    long duration = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DURATION));
                    long Size = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.SIZE));
                    VideoItem videoItem = new VideoItem(id,duration,Size,album,artist,displayName,title,mime_type,path);
                    list.add(videoItem);
                }
                cursor.close();
            }
        }
        return list;
    }
}
```

# 完成视频列表
>开始视频列表主界面的主体工程，将前面的视频列表的部件组装成完整的视频播放列表

VideoListActivity.java

```
public class VideoListActivity extends AppCompatActivity {
    private static final String TAG = "VideoListActivity";
    ListView mVideoList;
    VideoListAdapter mVideoListAdapter;
    List<VideoItem> videoItems;
    int VideoSize;
    @Override
    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_videolist);
        AbstractProvider provider = new VideoProvider(this);
        videoItems = provider.getList();
        VideoSize = videoItems.size();
        mVideoListAdapter = new VideoListAdapter(VideoListActivity.this,videoItems);
        mVideoList = (ListView) findViewById(R.id.video_list);
        Log.i(TAG, "onCreate: "+ VideoSize);
        mVideoList.setAdapter(mVideoListAdapter);
        mVideoList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                /*DoSomething*/
            }
        });
        Refresh_thumbnailListView();
    }

    /*刷新视频缩略图列表*/
    private void Refresh_thumbnailListView() {
        final Object data = getLastCustomNonConfigurationInstance();
        if(data == null){
            new LoadImagesFormSDCard().execute();
        }else {
            final LoadImage[] thumbnails = (LoadImage[]) data;
            if (thumbnails.length == 0){
                new LoadImagesFormSDCard().execute();
            }
            for (LoadImage thumbnail:thumbnails){
                addImage(thumbnail);
            }
        }

    }

    private class LoadImagesFormSDCard extends AsyncTask<Object,LoadImage,Object>{

        @Override
        protected Object doInBackground(Object... params) {
            Bitmap bitmap;
            for (int i = 0; i < VideoSize; i++) {
                bitmap = getVideoThumbnail(videoItems.get(i).getPath(),150,150, Thumbnails.MINI_KIND);
                if(bitmap != null){
                    publishProgress(new LoadImage(bitmap));
                }
            }
            return null;
        }
        @Override
        protected void onProgressUpdate(LoadImage... values) {
            addImage(values);
        }
    }

    /**
     * 获取视频缩略图
     */
    private Bitmap getVideoThumbnail(String VideoPath,int width,int height,int kind){
        Bitmap bitmap;
        bitmap = ThumbnailUtils.createVideoThumbnail(VideoPath,kind);
        bitmap = ThumbnailUtils.extractThumbnail(bitmap,width,height,ThumbnailUtils.OPTIONS_RECYCLE_INPUT);
        return bitmap;
    }

    private void addImage(LoadImage... value) {
        for(LoadImage image : value){
            mVideoListAdapter.addthumbnail(image);
            mVideoListAdapter.notifyDataSetChanged();
        }
    }

    @Override
    public Object onRetainCustomNonConfigurationInstance() {
        final ListView grid = mVideoList;
        final int count = grid.getChildCount();
        final LoadImage[] list = new LoadImage[count];

        for (int i = 0; i < count; i++) {
            final ImageView v = (ImageView) grid.getChildAt(i);
            list[i] = new LoadImage(
                    ((BitmapDrawable) v.getDrawable()).getBitmap());
        }
        return list;
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
}

```

完成后的主界面效果图：

<center>
![视频列表效果](http://of6x0sb2r.bkt.clouddn.com/videolist.png "视频列表效果")
</center>
