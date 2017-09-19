##8.17
###路由

>laravel给我最大的惊喜就是路由功能

* 路由可以直接输出视图（不规范）
* 一般用路由指定控制器，让控制器输出视图
* 路由的参数名字可以随意，控制器只识别该参数的顺序（第一个，第二个），填了参数后，uri必须带参数，否则报错，除非参数后加 `？` 并且在控制器接收参数时间默认定义为空，
* 路由可以追加对参数输入控制（正则）；

		Route::get('test/{id?}/{name?}','TestController@testFunction');
###数据库
* DB facade
>使用原生语句

		DB::select('select * from student');
		DB::insert('insert into student(id,name) values(?,?)',['jiawei',18]);
		DB::update('update student set name = ? where id = ?',[1,'jiawei2'])
		DB::delete('delete from student where id = ?',[1]);

* 查询构造器

		DB::table('student')->insert(['name'=>'jiawei']);//插入
		DB::table('student')->where('id',12)->update(['name'=>'jiawei2']);//更新
		DB::table('student')->increment('id',1);//id自增1，可以带条件where
		DB::table('student')->decrement('id',1,['name'=>'jiawei3']);//id自减1，同时将name字段修改为jiawei3，可以带条件where
		DB::table('student')->where('id','>=',2)->delete();//删除id大于等于2的数据
		DB::table('student')->truncate();//清空数据表
		DB::table('student')->whereRaw('id >= ? and age > ?',[1,18])->get();//查询所有,where多条件查询
		DB::table('student')->orderBy('id','desc')->first();//查询一条
		DB::table('student')->pluck('name');//查询name字段所有
		DB::table('student')->lists('name','id');//查询name字段，并用id作为键值
		DB::table('student')->select('id,name')->get();//指定查找id,name字段
		DB::table('student')->chunk(1000,function($res){
			dd($res);
			//return false 停止语句
		});//分段查询，每段1000条数据

		//聚合函数
		count() //统计
		max() //最大值
		min() //最小值
		avg() //平均数
		sum() //和
		
* ORM
>将模型名称于数据表对应
	
		//模型
		class studebt extends Model{
			protected $table = 'student';//指定表
			protected $primaryKey = 'id';//指定主键
			Protected $fillable = ['name','id'];//指定允许批量赋值的字段
			Protected $guarded = ['name','id'];//指定不允许批量赋值的字段
			public $timestamps =true;//自动维护时间戳		
			protected function getDataFormat(){//维护时间格式改为时间戳，默认为年份
				return time();
			}
			protected function asDataTime($val){//时间戳输出格式
				echo data('Y-m-d H:i:s');
			}
		}

		//使用模型查询数据
		Student::all() //查询所有
		Student::find(1)//根据主键查找，查询ID为1
		Student::findOrFail()//根据主键查找，找不到则报错
		Student::get() //查询所有
		Student::where('id','>=',1)->order('id','desc')->first();//查询一条
		Student::chunk(2,function($student){
			dd($student);
		})//分段查询
		
		//使用模型新增数据
		$student = new Student();
		$stduent->name='jiawei3';
		$student->save()//更新（时间字段会自动填充）

		//使用模型Create方法新增数据
		Student::Create(['name'='jiawei4'])//批量赋值
		
		firstOrCreate()//查找一条，没有则新增
		firstOrNew()//查找一条，没有则建立实例，想添加则执行save()
		
		//使用模型修改数据
		$student = Student::find(1);//实例
		$student->name='jiawei5';
		$student-> save();
		Student::where('id','>=',1)->update(['name'=>jiawei6]);//批量（结合查询）

		//使用ORM删除数据
		$studebt = Student::find(1);//实例
		$student->delete();//模型删除
		Student::destroy([1,2]);//删除id为1，id为2
		Student::where('id','>',1)->delete();//
		
		
		//聚合函数
		Student::count()//统计
		Student::max()//最大值
		
		
###blade模板引擎
* 模板继承

		//定义父模板layouts
		@section('header') //section
		@show
		
		@yield('content','主要内容区域')//yiekd
		
		//子模板继承
		@extends('layouts')

		@section('header')//header
		@parent//引用父模板内容
		@stop

		@section('content')//content
		@stop

		//定义子视图
		
		@include('student/common',['message'=>'传递参数到子视图'])//在视图中引用
* 流程控制
	
		@if() 
		@elseif() 
		@else
		@endif
		
		@unless//if取反
		@endunless
		
		@for()
		@endfor

		@foreach()
		@endforeach()

		@forelse()//类foreach
		@empty
		@endforelse

* 语法
		<!--html注释-->
		{{--模板注释，浏览器控制台不可见--}}

*模板中的url
		
		url('url')//url为路由中定义的别名
		action('studentConotroller@url')
		route('url')//url为路由中定义的别名

##8.18
###controller之request
>//Resqest类依赖注入

		//使用symfony框架的httpFoundtion组件，将请求数据存放在$data中	
		public function test(Request $data){
			$data->input('name','参数未定义');//值，默认值
			$data->all();//所有
			$data->method();//当前请求类型
			$data->isMethod('GET');//判断是否为GET请求
			$data->ajax();//判断是否为ajax请求
			$data->is('student/*');判断当前控制器
			$data->url()//当前url
			$daat->has()//判断输入值是否存在
		}

###controller之session
>在两个控制器中启用（sessionStart）中间件<br>
>  Route::group(['middleware'=>['web'],function(){
>       Route::any('sessionPut','Student@sessionPut');
>       Route::any('sessionGet','Student@sessionGet');
>  }])
		
		//http request类的session方法
		$request->session()->put('key','value');
		$request->session()->get('key');
		
		//session()辅助函数
		session()->put('key','value');
		session()->get('key');
		
		//Session类
		Session::put('key','value');
		Session::put('key'=>'value');
		Session::get('key','default');//为空则输出默认值
		
		//储存数组
		Session::push('key','value1');
		Session::push('key','value2');

		Session::pull('key');//取出并删除
		Session::all();//取出所有值
		Session::has('key')//是否存在
		Session::forget('key')//删除
		Session::flush()//删除所有
		Session::flash()//暂存，访问完后就删除

###controller之Response
		
		//返回json数据
		response()->json($data);//将数组转化为json

		//重定向
		redirect('session2')->with('msg','快闪数据');//在session2中用Session::get('msg')获取数据，访问后删除（暂存）；
		redirect()->action('Student@session2')->with('msg','快闪数据');//在session2中用Session::get('msg')获取数据，访问后删除（暂存）；
		redirect()->route('url')//别名
		redirect()->back();返回上一个页面

###controller之Middleware
* 新建
		namespace App\Http\Middleware;

		use Closure;
		
		class Activity
		{
		    public function handle($request,Closure $next){
		        if(time() < strtotime('2017-8-17')){
		           return redirect('activity0');
		        }
		       return $next($request);
		    }
		}
* 注册

		在kernel.php中的routeMiddleware加入
		'activity' => \App\Http\Middleware\Activity::class,
* 使用

		//在路由中指定
		Route::group(//将中间件分配到路由activity中
		['middleware'=>['activity']]
		,function(){
	    Route::any('activity1','TestController@activity1');
		});
		
		Route::any('activity0','TestController@activity0');

* 前置后置操作

		页面跳转前执行//前置
		页面跳转后执行//后置
###表单 

		
		//分页
		Student::paginate(10)//10条一页
		{{$student->render()}}//模板分页输出		

		1.实现添加成功后暂存添加信息到session必须加入web群组，所以csrf也会被启用//方式XSS攻击的手段
		须在表单模板出加上 {{csrf_field}}，否则提交时会报错 //创建一个包含token的input随着表单提交
		
		2.控制器表单验证
		//原理通过web中间件群组的ShareErrorsFronSEssion组，
		这个组把错误信息储存在全局变量中,分享到提表单页面
		$error->first()//打印第一条错误信息
		$error->all()//打印所有错误信息
		
		$this->validate($request,['student,name'=>request|min:2|max:20,
		 'student,age'=>request|integer,
		 'student,sex'=>request|integer,
		],['request'=>':attribute 为必填项'，
		   'integer'=>':attribute 必须为整型'，
		   'min'=>':attribute 最少两个字符'，
		   'max'=>':attribute 最多20个字符'，
		],['Student.name'=>'姓名',
		   'Student.sex'=>'性别',
		   'Student.age'=>'年龄',
			])		
		
		//

## 8.26

###curl
		//下载
		$ch = curl_init();//初始化
		curl_setopt($ch,CURLOPT_URL,$url);//设置url
		curl_setopt($ch,CURLOPT_TIMEOUT,$timeout);//设置超时时间
		curl_setopt($ch,RETURNTRANSFER,true);//不直接打印
		$file = fopen($dir,'wb');//打开或新建文件，并设定写入状态
		curl_setopt($ch,CURLOPT_FILE,$file);//将爬取到的资源保存到文件中
		$temp = curl_exec();//执行curl
		fclose($file);//关闭文件

		//上传
		$ch = curl_init();//初始化
		curl_setopt($ch,CURLOPT_URL,$url);//设置url
		curl_setopt($ch,CURLOPT_HEADER,0);//不设置网页头部HEADER
		curl_setopt($ch,RETURNTRANSFER,true);//不直接打印
		curl_setopt($ch,CURLOPT_TIMEOUT,$timeout);//设置超时时间
		curl_setopt($ch,CURLOPT_USERPWD,'jiaweiaa:123456');//账号：密码
		curl_setopt($ch,CURLOPT_UPLOAD,1);//上传
		$file = fopen($dir,'r');//打开文件
		curl_setopt($ch,CURLOPT_INFILE,$file);//上传指定文件
		curl_setopt($ch,CURLOPT_INFILESIZE,filesize($file));//设置文件大小
		$temp = curl_exec();//执行curl
		fclose($file);//关闭文件

		//执行post
		$ch = curl_init();//初始化
		curl_setopt($ch,CURLOPT_URL,$url);//设置url
		curl_setopt($ch,RETURNTRANSFER,true);//不直接打印
		curl_setopt($ch,CURLOPT_HEADER,0);//不设置网页头部HEADER
		curl_setopt($ch,CURLOPT_POST,1);//设置post
		curl_setopt($ch,CURLOPT_POSTFIELDS,'name=jiawei&id=1');//请求参数
		($ch,CURLOPT_HTTPHEADER,array("application/x-www-form-urlencode;charset=utf-8","Content-length: ".strlen('name=jiawei&id=1')));//请求头部以及参数长度
		$temp = curl_exec();//执行curl
	
		

		  
		
		