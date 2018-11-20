---
layout: posts
title: 【DataTable】关于实现DataTable后端分页过程中的一些问题总结
date: 2018-11-20 14:48:49
tags:
- DataTable
categories: 问题解决
---

# 一、场景

&ensp;&ensp;&ensp;&ensp;公司新开发的一个web项目，项目中一个功能是从失败交易流水表中按日期查询失败的交易，以列表的形式展示出来。前端列表使用了DataTable，DataTable自带前端分页和后端分页。所谓前端分页就是一次性从数据库中查出所有数据返回给前端，前端自动进行分页。这种处理方式在数据量较小的情况下还可以，当数据量较大（具体数据量没有测试）会导致前端加载数据缓慢卡顿，同时因为后端一次性从数据库查出大量数据放在内存中，会导致内存资源消耗过大而卡顿甚至宕机。为了解决这个瓶颈问题，采用后端分页的形式。后端分页即前端传递当前页码以及当前页显示的数据量给后端，后端只查询当前页要进行展示的数据。从而避开了由于数据量过大而导致的卡顿问题。

# 二、基础代码

&ensp;&ensp;&ensp;&ensp;DataTable默认的分页机制为前端分页，此处不过多陈述，通过查找资料了解到，后端分页需要将原来的

```java
bServerSide:false
```

修改为：

```java
bServerSide:true
```

后端分页对返回的JSON数据格式有要求，具体格式如下：

```java
{"sEcho":"1","iTotalRecords":"0","iTotalDisplayRecords":"0","aaData":[]}
```

&ensp;&ensp;&ensp;&ensp;其中sEcho是前端传递的，只需获取然后原数返回即可，iTotalRecords字面理解意思是当前数据表中总的记录数，iTotalDisplayRecords是当前页面要展示的记录数，aaData为返回列表的数据。

&ensp;&ensp;&ensp;&ensp;这里先展示一版基础代码，也就是我从网上查找资料写的代码，通过基础代码，后边一步一步的发现问题解决问题。

HTML部分：

```html
</div>
       <div class="mr-20 ml-20">
           <table id="trafficMonitorFailed" class="table table-border table-bordered table-hover table-bg table-sort">
                   <thead>
                       <tr class="text-c">
                           <th>交易日期</th>
                           <th>交易时间</th>
                           <th>交易流水号</th>
                           <th>交易唯一标识</th>
                           <th>渠道</th>
                           <th>交易响应时间</th>
                           <th>交易分类</th>
                           <th>交易编码</th>
                           <th>交易描述</th>
                           <th>错误码</th>
                           <th>错误描述</th>
                       </tr>
                   </thead>
               </table>
        </div>
```

JS部分：

```javascript
function dataTableDraw(){
 $("#trafficMonitorFailed").dataTable({
  bServerSide:true, //开启后端分页
  bDestroy: true,         //下边两个属性应该是重载数据相关的 不加在加载数据会弹窗报错 点击确定后显示数据
  bRetrieve: true,
  bProcessing: true, //显示加载数据时的提示
  bInfo:true,  //显示信息 如 当前x页 共x条数据等
  bSort:true,  //允许排序
  bFilter:true,  //检索、筛选框
  sAjaxSource: rootUrl + "getTrafficMonitorFailedList", //请求url
  bLengthChange:true, //支持变更页面显示数据行数
  sPaginationType: "bootstrap", //翻页风格
  bPaginate:true,  //显示翻页按钮
  fnServerData: retrieveData, //执行函数
  aoColumns:[//列表元素  支持多种属性 
                   { "mData": "tranDate","fnRender":function(data,val){
        var JSONDate = new Date(val.time); 
        return JSONDate.format('yyyy-MM-dd');
       },"width":"200px"},
                   { "mData": "tranTime","fnRender":function(data,val){
        var JSONTime = new Date(val.time);
        return JSONTime.format('yyyy-MM-dd HH:mm:ss');
       }},
                   { "mData": "seqNo"},
                   { "mData": "platformId"},
                   { "mData": "channel"},
                   { "mData": "costTime"},
                   { "mData": "reqType"},
                   { "mData": "reqInterface"},
                   { "mData": "reqDesc","bSortable":false},  //不允许当前页排序
                   { "mData": "retCode","bSortable":false},
                   { "mData": "retMsg","bSortable":false}
               ],
      oLanguage: {  
       "sProcessing" : "正在加载中......",  
       "sLengthMenu" : "_MENU_",  
       "sZeroRecords" : "无记录",  
       "sEmptyTable" : "表中无数据存在！",  
       "sInfo" : "当前显示 _START_ 到 _END_ 条，共 _MAX_  条记录",  
       "sInfoEmpty" : "没有数据",  
       "sInfoFiltered" : "数据表中共为 _TOTAL_ 条记录",  
       "sSearch" : " ",  
       "oPaginate" : {  
        "sFirst" : " 首页 ",  
        "sPrevious" : " 上一页 ",  
        "sNext" : " 下一页 ",  
        "sLast" : " 末页 "  
        }  
 }  
 });
 $(".dataTables_wrapper .dataTables_filter input").attr("placeholder","检索内容");
}
//对应上边的回调函数 参数个数不变 名字可改 第一个为请求url  第二个为上送数据 第三个为回调函数
function retrieveData(sSource,aoData,fnCallback) {
 var startDate = {
   "name":"startDate",
   "value":$("#from").val()
 }
 var endDate = {
   "name":"endDate",
   "value":$("#to").val()
 }
 //我这里按照请求数据的格式增加了自己的查询条件 请求数据格式固定为 name-value的格式 可以使用
 //alert打印查看 包含了基本的页码、页面数据元素、等信息以及新增的查询条件
 aoData.push(startDate);
 aoData.push(endDate);
 $.ajax({
     url : sSource,//这个就是请求地址对应sAjaxSource
     data : {"aoData":JSON.stringify(aoData)},//这个是把datatable的一些基本数据传给后台,比如起始位置,每页显示的行数
     type : 'post',
     dataType : 'json',
     async : false,
     success : function(result) {
         fnCallback(result);//把返回的数据传给这个方法就可以了,datatable会自动绑定数据的
     },
     error : function(msg) {
     }
 });
}
```

后端Controller层：

```java
	@RequestMapping(value="/page/getTrafficMonitorFailedList",method=RequestMethod.POST)
	@ResponseBody
	public String getTrafficMonitorFailedList(String aoData){
		List<TrafficMonitorFailed> trafficMonitorFailed = new ArrayList<TrafficMonitorFailed>();
        JSONArray jsonarray=(JSONArray) JSONArray.fromObject(aoData);//json格式化用的是fastjson
        if(jsonarray == null||jsonarray.size() ==0){
           return WebFactory.createErrorResponse("错误的查询条件");
        }
        String startDate = formatDate.format(new Date());
        String endDate = formatDate.format(new Date());
        String sEcho = null;
        int iDisplayStart = 0; // 起始索引
        int iDisplayLength = 10; // 每页显示的行数
        int count = 0;
        for (int i = 0; i < jsonarray.size(); i++) {
            JSONObject obj = (JSONObject) jsonarray.get(i);
            if (obj.get("name").equals("sEcho"))
                sEcho = obj.get("value").toString();
 
            if (obj.get("name").equals("iDisplayStart"))
                iDisplayStart =Integer.parseInt(obj.get("value").toString());
 
            if (obj.get("name").equals("iDisplayLength"))
                iDisplayLength = Integer.parseInt(obj.get("value").toString());
            
            if (obj.get("name").equals("startDate"))
            	startDate = obj.get("value").toString();
            
            if (obj.get("name").equals("endDate"))
            	endDate = obj.get("value").toString();
            
        }
        
            trafficMonitorFailed = this.trafficMonitorFailedService.getTrafficMonitorFailedListSearch(startDate, endDate, iDisplayStart, iDisplayLength);
            count = this.trafficMonitorFailedService.getTrafficMonitorFailedListCountSearch(startDate, endDate).getTotal();
        
        JSONObject getObj = new JSONObject();
        getObj.put("sEcho", sEcho);// DataTable前台必须要的
        getObj.put("iTotalRecords",count);
        getObj.put("iTotalDisplayRecords",trafficMonitorFailed.size());
        getObj.put("aaData", trafficMonitorFailed);//把查到数据装入aaData,要以JSON格式返回
        return getObj.toString();
 
	}
```

第一版的基础代码如上，其中有一些如数据Model的代码，不同业务场景Model不同，此处不做列举。

# 三、问题描述/解决

## Q1.Cannot read property 'length' of undefined

&ensp;&ensp;&ensp;&ensp;上述代码运行之后，页面加载之后一直显示加载中，F12到调试模式可以看到控制台报错“Cannot read property 'length of undefined'”，而且这个是jquery报的错，可以说是非常恼火了。经过分析，可能是返回的数据格式不对，因此在js的回调函数中进行了修改，将JSON字符串转换成JSON数据的格式，修改后如下：

```javascript
 $.ajax({
     url : sSource,//这个就是请求地址对应sAjaxSource
     data : {"aoData":JSON.stringify(aoData)},//这个是把datatable的一些基本数据传给后台,比如起始位置,每页显示的行数
     type : 'post',
     dataType : 'json',
     async : false,
     success : function(result) {
    	 var ret = eval("(" + result + ")");
         fnCallback(ret);//把返回的数据传给这个方法就可以了,datatable会自动绑定数据的
     },
     error : function(msg) {
     }
 });
```

修改之后运行，此问题解决

## Q2.DataTables warning (table id = 'trafficMonitorFailed'):Requested unknown parameter '0' from the data source for row 0

&ensp;&ensp;&ensp;&ensp;上述代码运行之后，页面弹窗报错“DataTables warning (table id ='trafficMonitorFailed'):Requested unknown parameter '0' from the data source for row 0”这个问题诈一看也是一脸蒙蔽，后来发现应该是dataTable版本的问题，需要将数据表中的mData修改为mDataProp,修改之后如下：

```java
                   { "mDataProp": "tranDate","fnRender":function(data,val){
					   var JSONDate = new Date(val.time);	
					   return JSONDate.format('yyyy-MM-dd');
				   },"width":"200px"},
                   { "mDataProp": "tranTime","fnRender":function(data,val){
					   var JSONTime = new Date(val.time);
					   return JSONTime.format('yyyy-MM-dd HH:mm:ss');
				   }},
                   { "mDataProp": "seqNo"},
                   { "mDataProp": "platformId"},
                   { "mDataProp": "channel"},
                   { "mDataProp": "costTime"},
                   { "mDataProp": "reqType"},
                   { "mDataProp": "reqInterface"},
                   { "mDataProp": "reqDesc","bSortable":false},
                   { "mDataProp": "retCode","bSortable":false},
                   { "mDataProp": "retMsg","bSortable":false}
```

修改之后运行，问题解决。

## Q3.数据明明很多，DataTable仅显示了一页的数据，并且没有显示其它页的按钮，不可以翻页。

&ensp;&ensp;&ensp;&ensp;这个问题的意思是，我数据库中有很多数据，F12调试也能看到返回了一页10条数据，并且iTotalRecords为32，按道理应该显示一页数据之后，可以进行翻页，总共显示4页，但是实际的页面却只显示了一页数据，列表下方的翻页处也仅仅有“1”的页签，没有其它的页签，上一页下一页都不能点击。这个问题困扰了我许久，后来私信网上一位大神给出了解决方案。原因是iTotalRecords和iTotalDisplayRecords数据放反了。这个我到现在也没有理解，按照字面意思itotalRecords确实是应该放总查询数据量，iTotalDisplayRecords为当前页要展示的数据量。而实际的使用过程中，这两个数据应该是反了过来。不知道其它人有没有遇到这种情况。
修改代码如下：

```java
getObj.put("iTotalRecords",trafficMonitorFailed.size());
getObj.put("iTotalDisplayRecords",count);
```

即将原来两个放置数据统计个数的值互换。运行之后，问题解决，心心念念的后端分页终于实现了。

## Q4.后端分页实现之后，检索筛选、排序功能失效。

&ensp;&ensp;&ensp;&ensp;实现了基本的后端分页，点击了几页试了一下，分页效果没问题，但是检索跟排序的功能都没有反应。于是F12开启调试模式，发现点击排序和在检索框输入文字之后，都发起了后台请求，因此可以判定，后端分页的检索跟排序功能需要后端代码实现。实现原理，在前端传递的aoData中，会储存要排序的列、排序的方式以及检索的内容。因此可以通过后端修改SQL的形式完成。
修改之后的Controller代码为:

```java
	@RequestMapping(value="/page/getTrafficMonitorFailedList",method=RequestMethod.POST)
	@ResponseBody
	public String getTrafficMonitorFailedList(String aoData){
		List<TrafficMonitorFailed> trafficMonitorFailed = new ArrayList<TrafficMonitorFailed>();
        JSONArray jsonarray=(JSONArray) JSONArray.fromObject(aoData);//json格式化用的是fastjson
        if(jsonarray == null||jsonarray.size() ==0){
           return WebFactory.createErrorResponse("错误的查询条件");
        }
        String startDate = formatDate.format(new Date());
        String endDate = formatDate.format(new Date());
        String sEcho = null;
        int iDisplayStart = 0; // 起始索引
        int iDisplayLength = 10; // 每页显示的行数
        int orderColumn = 0;//默认排序列
        String orderDir = "asc";//默认排序方式为升序
        String sSearch = "";//默认检索内容
        int count = 0;
        for (int i = 0; i < jsonarray.size(); i++) {
            JSONObject obj = (JSONObject) jsonarray.get(i);
            if (obj.get("name").equals("sEcho"))
                sEcho = obj.get("value").toString();
 
            if (obj.get("name").equals("iDisplayStart"))
                iDisplayStart =Integer.parseInt(obj.get("value").toString());
 
            if (obj.get("name").equals("iDisplayLength"))
                iDisplayLength = Integer.parseInt(obj.get("value").toString());
            
            if (obj.get("name").equals("startDate"))
            	startDate = obj.get("value").toString();
            
            if (obj.get("name").equals("endDate"))
            	endDate = obj.get("value").toString();
            
            if (obj.get("name").equals("iSortCol_0"))
            	orderColumn = Integer.parseInt(obj.get("value").toString());
            
            if (obj.get("name").equals("sSortDir_0"))
            	orderDir = obj.get("value").toString();
            
            if (obj.get("name").equals("sSearch"))
            	sSearch = obj.get("value").toString();
        }
        
        if("".equals(sSearch)||sSearch == null){
            trafficMonitorFailed = this.trafficMonitorFailedService.getTrafficMonitorFailedList(startDate, endDate, iDisplayStart, iDisplayLength, orderColumn, orderDir);
            count = this.trafficMonitorFailedService.getTrafficMonitorFailedListCount(startDate, endDate).getTotal();
        }else{
            trafficMonitorFailed = this.trafficMonitorFailedService.getTrafficMonitorFailedListSearch(startDate, endDate, iDisplayStart, iDisplayLength, orderColumn, orderDir,sSearch);
            count = this.trafficMonitorFailedService.getTrafficMonitorFailedListCountSearch(startDate, endDate,sSearch).getTotal();
        }
        JSONObject getObj = new JSONObject();
        getObj.put("sEcho", sEcho);// DataTable前台必须要的
        getObj.put("iTotalRecords",trafficMonitorFailed.size());//显示的行数,这个要和上面写的一样
        getObj.put("iTotalDisplayRecords",count);//总行数
        getObj.put("aaData", trafficMonitorFailed);//把查到数据装入aaData,要以JSON格式返回
        return getObj.toString();
 
	}
```

&ensp;&ensp;&ensp;&ensp;这里前端传递的orderColumn是int类型，标记的是列的序号，SQL语句中可以判断序号然后ORDER BY指定的列，orderDir传递的是排序方式有两种一种是ASC另一种是DESC。sSearch传递的是检索框的内容，这里为了避免没有检索的情况还使用like语句模糊查找而影响效率，没有想到其它办法，单单的使用了判断sSearch如果为空则不使用like语句查找。多说一句，分页查询使用的sql语句我这里使用了limit a，b的形式，从a+1位置查，查b条数据。这里忘了贴SQL语句，大概的形式就是select xxx from xxx where xxx order by xxx asc/desc limit a,b 这种，如果后期数据量大了出现瓶颈再继续优化。

## Q5.给DataTable增加横向滚动条

&ensp;&ensp;&ensp;&ensp;为了美观，我想保证每一条数据都在一行上显示，不进行换行。然后就会出现数据过长，一页显示不下的情况，因此需要增加滚动条。网上查到不同版本使用的形式不同，我这里是使用如下形式

```java
sScrollX:"100%"
```

&ensp;&ensp;&ensp;&ensp;还有另一种方式是将“100%”改为true的形式。增加滚动条还需要设置文本框的数据处于一行显示而不是自动换行。

修改html代码如下：

```html
</div>
       <div class="mr-20 ml-20">
           <table id="trafficMonitorFailed" class="table table-border table-bordered table-hover table-bg table-sort" style="white-space:nowrap">
                   <thead>
                       <tr class="text-c">
                        	<th>交易日期</th>
                           <th>交易时间</th>
                           <th>交易流水号</th>
                           <th>交易唯一标识</th>
                           <th>渠道</th>
                           <th>交易响应时间</th>
                           <th>交易分类</th>
                           <th>交易编码</th>
                           <th>交易描述</th>
                           <th>错误码</th>
                           <th>错误描述</th>
                       </tr>
                   </thead>
               </table>
        </div>
```

即增加了样式：

```java
style="white-space:nowrap"
```

&ensp;&ensp;&ensp;&ensp;至此，后端分页的功能全部实现。如有不足，还望指正。