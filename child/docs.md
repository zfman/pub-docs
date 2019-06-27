## 接口数据格式
> 为别人写的一款早教课表软件的请求接口格式

### 添加课程

提交参数：
```
    name:课程名
    teacher:教师
    room:教室
    weeks，上课周次，以空格分隔，如:1 3 4 5 6
    day:星期，整数，如1表示星期一
    start:起始节次，整数，如3表示第三小节开始上课
    step:持续节次，整数，如2表示持续两节
```

响应结果：
```json
{
	"code":200,
	"msg":"成功",
	"data":{
		"courseId":1
	}
}
```

### 删除课程

提交参数：
```
    courseId:课程ID
```

响应结果：
```json
{
	"code":200,
	"msg":"成功",
	"data":{}
}
```

### 编辑课程

提交参数：
```
    courseId:课程ID
    name:课程名
    teacher:教师
    room:教室
    weeks，上课周次，以空格分隔，如:1 3 4 5 6
    day:星期，整数，如1表示星期一
    start:起始节次，整数，如3表示第三小节开始上课
    step:持续节次，整数，如2表示持续两节
```

响应结果：
```json
{
	"code":200,
	"msg":"成功",
	"data":{}
}
```

### 获取该用户课程

响应结果：
```json
{
    "code":200,
    "msg":"成功",
    "data":[
        {
            "courseId":1,
            "name":"计算机在化学中的应用",
            "room":"理化楼207",
            "teacher":"赵雷",
            "weeks":"1 3 6 7 8 9",
            "start":1,
            "step":3,
            "day":2
        },
        {
            "courseId":2,
            "name":"高等数学",
            "room":"教学楼201",
            "teacher":"赵六",
            "weeks":"1 3 6 10 11 12 13 14 15",
            "start":3,
            "step":5,
            "day":2
        }
    ]
}
```

## 包结构

```
    activity: 视图层
    api: 接口层
        constants: 接口用到的常量，接口地址等信息
        model: 接口响应数据格式
        service: 接口的参数、响应结果定义规范
        ApiUtils.java: 请求工具类，初始化Retrofit
        TimetableRequest.java: 请求类
    fragment: 也是视图层
        FuncFragment: 主页面左侧的页面
        ScheduleFragment: 主页面右侧的页面
    other: 杂项
        event: 事件总线Event用到的类
        tools: 工具类
        AddModel.java: 添加课程时用来存储每个时间段的信息
        ScheduleDao.java: 工具类
        ShareConstants.java: 存储到本地的键常量
    LoginActivity.java: 登陆页面
    MyApplication.java: 在Application中对一些第三库初始化
```

## 网络请求

现在的网络请求数据用的是[RAP](http://rap2.taobao.org/)模拟的，你需要修改api/constants包下的UrlContacts.java 文件
```java
public class UrlContacts {

    //模拟地址
    public final static String URL_BASE="http://rap2api.taobao.org/app/mock/";

    //获取课程
    public final static String URL_GET_COURSES="176384/child/getCourses";

    //删除课程
    public final static String URL_DELETE_COURSE="176384/child/deleteCourse";

    //添加课程
    public final static String URL_ADD_COURSE="176384/child/addCourse";

    //修改课程
    public final static String URL_MODIFY_COURSE="176384/child/modifyCourse";
}
```

网络请求参见 api包下代码

- 定义Service

指定参数以及返回的数据类型
```java
public interface TimetableService {

    @POST(UrlContacts.URL_GET_COURSES)
    Call<ListResult<CourseModel>> getCourses();

    @POST(UrlContacts.URL_DELETE_COURSE)
    @FormUrlEncoded
    Call<BaseResult> deleteCourse(@Field("courseId") int courseId);

    @POST(UrlContacts.URL_ADD_COURSE)
    @FormUrlEncoded
    Call<ObjResult<AddCourseResult>> addCourse(@Field("name") String name,
                                               @Field("teacher") String teacher,
                                               @Field("room") String room,
                                               @Field("weeks") String weeks,
                                               @Field("day") int day,
                                               @Field("step") int step,
                                               @Field("start") int start);

    @POST(UrlContacts.URL_MODIFY_COURSE)
    @FormUrlEncoded
    Call<BaseResult> modifyCourse(@Field("courseId") int courseId,
                                                  @Field("name") String name,
                                                  @Field("teacher") String teacher,
                                                  @Field("room") String room,
                                                  @Field("weeks") String weeks,
                                                  @Field("day") int day,
                                                  @Field("step") int step,
                                                  @Field("start") int start);
}

```

- 请求

```java
public class TimetableRequest {
    public static void getCourses(Context context, Callback<ListResult<CourseModel>> callback) {
        TimetableService timetableService = ApiUtils.getRetrofit(context)
                .create(TimetableService.class);
        Call<ListResult<CourseModel>> call = timetableService.getCourses();
        call.enqueue(callback);
    }

    public static void deleteCourse(Context context,int courseId,Callback<BaseResult> callback) {
        TimetableService timetableService = ApiUtils.getRetrofit(context)
                .create(TimetableService.class);
        Call<BaseResult> call = timetableService.deleteCourse(courseId);
        call.enqueue(callback);
    }

    public static void addCourse(Context context,String name,String teacher,
                                 String room,String weeks,int day,
                                 int step,int start,Callback<ObjResult<AddCourseResult>> callback) {
        TimetableService timetableService = ApiUtils.getRetrofit(context)
                .create(TimetableService.class);
        Call<ObjResult<AddCourseResult>> call = timetableService.addCourse(name,teacher,room,weeks,day,step,start);
        call.enqueue(callback);
    }

    public static void modifyCourse(Context context,int courseId,String name,String teacher,
                                 String room,String weeks,int day,
                                 int step,int start,Callback<BaseResult> callback) {
        TimetableService timetableService = ApiUtils.getRetrofit(context)
                .create(TimetableService.class);
        Call<BaseResult> call = timetableService.modifyCourse(courseId,name,teacher,room,weeks,day,step,start);
        call.enqueue(callback);
    }
}

```

- 使用

**获取课程，参见LoginActivity#getCourses() **

```java
public void getCourses(){
        TimetableRequest.getCourses(this, new Callback<ListResult<CourseModel>>() {
            @Override
            public void onResponse(Call<ListResult<CourseModel>> call, Response<ListResult<CourseModel>> response) {
                ListResult<CourseModel> list=response.body();
                if (list != null) {
                    if(list.getCode()==200){
                        List<CourseModel> courseModels=list.getData();
                        List<TimetableModel> timetableModels=new ArrayList<>();
                        int id0 = ScheduleDao.getApplyScheduleId(LoginActivity.this);
                        ScheduleName scheduleName = DataSupport.find(ScheduleName.class,id0);
                        if (scheduleName == null){
                            check();
                            id0 = ScheduleDao.getApplyScheduleId(LoginActivity.this);
                            scheduleName = DataSupport.find(ScheduleName.class,id0);
                        }

                        if(courseModels!=null){
                            for(CourseModel itemModel:courseModels){
                                TimetableModel saveModel=itemModel.toTimetableModel();
                                saveModel.setScheduleName(scheduleName);
                                timetableModels.add(saveModel);
                            }
                        }
                        DataSupport.saveAll(timetableModels);
                        loadingLayout.setVisibility(View.GONE);
                        ActivityTools.toActivityWithout(LoginActivity.this, MainActivity.class);
                        finish();
                    }else{
                        loadingLayout.setVisibility(View.GONE);
                        ToastTools.show(LoginActivity.this,list.getMsg()==null?"Msg is null":list.getMsg());
                    }
                }else{
                    loadingLayout.setVisibility(View.GONE);
                }
            }

            @Override
            public void onFailure(Call<ListResult<CourseModel>> call, Throwable t) {
                ToastTools.show(LoginActivity.this,t.getMessage());
                loadingLayout.setVisibility(View.GONE);
            }
        });
    }
```

**获取课程，参见ScheduleDao#delete **

```java
public static void delete(final Activity context, Schedule schedule) {
        if (schedule != null) {
            int id = (int) schedule.getExtras().get(TimetableModel.EXTRA_ID);
            final TimetableModel model = DataSupport.find(TimetableModel.class, id);
            if (model != null) {
                if(model.getCourseId()==0){
                    Toasty.error(context, "删除失败,courseId不存在！").show();
                    return;
                }
                TimetableRequest.deleteCourse(context, model.getCourseId(), new Callback<BaseResult>() {
                    @Override
                    public void onResponse(Call<BaseResult> call, Response<BaseResult> response) {
                        BaseResult result=response.body();
                        if(result!=null){
                            if(result.getCode()==200){
                                model.delete();
                                ShareTools.put(context, "course_update", 1);
                                Toasty.success(context, "删除成功！").show();
                                EventBus.getDefault().post(new UpdateScheduleEvent());
                                context.finish();
                            }else{
                                ToastTools.show(context,result.getMsg()==null?"Msg is null,code is "+result.getCode():result.getMsg());
                            }
                        }else{
                            ToastTools.show(context,"Error:body is null");
                        }
                    }

                    @Override
                    public void onFailure(Call<BaseResult> call, Throwable t) {
                        ToastTools.show(context,t.getMessage());
                    }
                });
            }
        }
    }
```

**添加课程，AddTimetableActivity#postDataToServer **

```java
private void postDataToServer(final List<TimetableModel> models) {
        if (models == null) return;
        final int[] index = {0};
        for (final TimetableModel model : models) {
            String weeks = "";
            for (Integer i : model.getWeekList()) {
                weeks += " " + i;
            }
            weeks = weeks.trim();
            TimetableRequest.addCourse(this, model.getName(), model.getTeacher(),
                    model.getRoom(),
                    weeks, model.getDay(),
                    model.getStep(),
                    model.getStart(),
                    new Callback<ObjResult<AddCourseResult>>() {
                        @Override
                        public void onResponse(Call<ObjResult<AddCourseResult>> call, Response<ObjResult<AddCourseResult>> response) {
                            synchronized (this) {
                                index[0]++;
                            }
                            ObjResult<AddCourseResult> result = response.body();
                            if (result != null) {
                                if (result.getCode() == 200) {
                                    AddCourseResult addCourseResult = result.getData();
                                    if (addCourseResult != null) {
                                        model.setCourseId(addCourseResult.getCourseId());
                                    } else {
                                        ToastTools.show(AddTimetableActivity.this, "Error:data is null");
                                    }
                                } else {
                                    ToastTools.show(AddTimetableActivity.this, result.getMsg() == null ? "Msg is null" : result.getMsg());
                                }
                            } else {
                                ToastTools.show(AddTimetableActivity.this, "Error:body is null");
                            }
                            //所有任务执行结束，保存到本地
                            if (index[0] == models.size()) {
                                DataSupport.saveAll(models);
                                Toasty.success(AddTimetableActivity.this, "保存成功", Toast.LENGTH_SHORT).show();
                                EventBus.getDefault().post(new UpdateScheduleEvent());
                                goBack();
                            }
                        }

                        @Override
                        public void onFailure(Call<ObjResult<AddCourseResult>> call, Throwable t) {
                            ToastTools.show(AddTimetableActivity.this, t.getMessage());
                        }
                    });
        }
    }
```

**修改课程，AddTimetableActivity#postDataToServerForModify **

```java
/**
     * 修改数据
     */
    private void postDataToServerForModify(final TimetableModel model) {
        if (model == null) return;
        String weeks = "";
        for (Integer i : model.getWeekList()) {
            weeks += " " + i;
        }
        weeks = weeks.trim();
        if(model.getCourseId()==0){
            Toasty.error(this, "修改失败,courseId不存在！").show();
            return;
        }
        TimetableRequest.modifyCourse(this, model.getCourseId(), model.getName(), model.getTeacher(),
                model.getRoom(),
                weeks, model.getDay(),
                model.getStep(),
                model.getStart(),
                new Callback<BaseResult>() {
                    @Override
                    public void onResponse(Call<BaseResult> call, Response<BaseResult> response) {
                        BaseResult result = response.body();
                        if (result != null) {
                            if (result.getCode() == 200) {
                                model.update(id);
                                ShareTools.putInt(AddTimetableActivity.this, "course_update", 1);
                                Toasty.success(AddTimetableActivity.this, "修改成功", Toast.LENGTH_SHORT).show();
                                EventBus.getDefault().post(new UpdateScheduleEvent());
                                goBack();
                            } else {
                                ToastTools.show(AddTimetableActivity.this, result.getMsg() == null ? "Msg is null" : result.getMsg());
                            }
                        } else {
                            ToastTools.show(AddTimetableActivity.this, "Error:body is null");
                        }
                    }

                    @Override
                    public void onFailure(Call<BaseResult> call, Throwable t) {
                        ToastTools.show(AddTimetableActivity.this, t.getMessage());
                    }
                });
    }
```


## 通知页面逻辑

参见 TimetableNotifyActivity#addCourseItemView

要求今日点击之后该课程在今日不再显示，具体逻辑如下：

- 不管点击取消还是确认，都向本地保存一个标志，比如课程名：高数,那么点击之后在首选项中保存"set_count_time_高数":"2019-05-15"
- 如果点击的是确认，则更新本地中保存的计数
- 进入页面时获取今日所有课程，然后过滤掉已经点击过的

```java
private void addCourseItemView(List<Schedule> list,ScheduleName scheduleName) {
		courseContainerLayout.removeAllViews();
		TextView nameTextView, roomTextView;
		if(list==null) return;
		for (int i = 0; i < list.size(); i++) {
			final Schedule course = list.get(i);
			//今天已经点击过了，过滤掉
			if(ScheduleDao.isTodayCountTime(this,course.getName())){
				continue;
			}
			final View view = LayoutInflater.from(this).inflate(R.layout.item_notify_layout, null);
			nameTextView = (TextView) view.findViewById(R.id.id_manager_name);
			roomTextView = (TextView) view.findViewById(R.id.id_manager_room);

			final TimetableModel model= (TimetableModel) course.getExtras().get(TimetableModel.EXTRA_MODEL);
			final TextView cancelTextView=view.findViewById(R.id.id_btn_cancel);
			TextView okTextView=view.findViewById(R.id.id_btn_ok);
			cancelTextView.setOnClickListener(new OnClickListener() {
						@Override
						public void onClick(View v) {
							if(model!=null){
								ScheduleDao.applySetCountTime(TimetableNotifyActivity.this,model.getName());
								view.setVisibility(View.GONE);
								ToastTools.show(TimetableNotifyActivity.this,"操作成功");
							}
						}
					});

			okTextView.setOnClickListener(new OnClickListener() {
						@Override
						public void onClick(View v) {
							if(model!=null){
								ScheduleDao.applySetCountTime(TimetableNotifyActivity.this,model.getName());
								ScheduleDao.addCount(TimetableNotifyActivity.this,model.getName());
								view.setVisibility(View.GONE);
								ToastTools.show(TimetableNotifyActivity.this,"操作成功");
							}
						}
					});

			nameTextView.setText(model.getName());
			roomTextView.setText("地点:"+model.getRoom());
			cancelTextView.setTextColor(Color.RED);
			okTextView.setTextColor(Color.BLUE);
			courseContainerLayout.addView(view);
		}
	}
```

ScheduleDao.java

```java
/**
     * 保存设置计数的时间
     * @param context
     */
    public static void applySetCountTime(Context context,String name){
        SharedPreferences sp=context.getSharedPreferences("notify_zfman",Context.MODE_PRIVATE);
        SharedPreferences.Editor editor=sp.edit();
        Date date=new Date();
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
        editor.putString("set_count_time_"+name,sdf.format(date));
        editor.commit();
    }

    /**
     * 判断保存的设置计数的时间是否为今天
     * @param context
     */
    public static boolean isTodayCountTime(Context context,String name){
        SharedPreferences sp=context.getSharedPreferences("notify_zfman",Context.MODE_PRIVATE);
        Date date=new Date();
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
        String now=sp.getString("set_count_time_"+name,null);
        if(now==null||!now.equals(sdf.format(date))){
            return false;
        }
        return true;
    }
```
