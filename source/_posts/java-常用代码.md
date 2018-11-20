---
title: java 常用代码
date: 2018-09-20 03:24:23
tags: 
categories: java
description: "java常用代码汇总，日期，接口调用，是否为空，读取配置文件"
---

## 读取配置文件

#### configuration2读取配置文件

```

import org.apache.commons.configuration2.Configuration;
import org.apache.commons.configuration2.FileBasedConfiguration;
import org.apache.commons.configuration2.PropertiesConfiguration;
import org.apache.commons.configuration2.builder.ReloadingFileBasedConfigurationBuilder;
import org.apache.commons.configuration2.builder.fluent.Parameters;
import org.apache.commons.configuration2.ex.ConfigurationException;

public static String getInclude() throws ConfigurationException {
	ReloadingFileBasedConfigurationBuilder<FileBasedConfiguration> 
	    include = new ReloadingFileBasedConfigurationBuilder<FileBasedConfiguration>(PropertiesConfiguration.class)
		.configure(new Parameters().properties().setEncoding("utf-8").setFileName("include.properties"));
	Configuration config = (Configuration)include.getConfiguration();
	String source = config.getString("ips");
	return source;
}

```
#### static读取配置文件

```
    public static Properties properties=new Properties();
    public static String DEFAULT_FILENAME="default.properties";
    加载配置文件
static {
    // 1. load library defaults
    InputStream in = demo.class.getResourceAsStream(DEFAULT_FILENAME);//类名.class

    if (in == null) {
        throw new RuntimeException(DEFAULT_FILENAME + " not found");
    } else {
    if (!(in instanceof BufferedInputStream))
        in = new BufferedInputStream(in);
        try {
        properties.load(in);
        in.close();
        } catch (Exception e) {
        throw new RuntimeException("Error while processing "
        + DEFAULT_FILENAME, e);
        }
    }

}

    /**
    * 获取prop值
    * @param key 输入的参数
    * @return 返回value
    */
    public static String getProperty(final String key) {
    return properties.getProperty(key);
    }

```


##  MD5加密

```
import org.apache.commons.codec.digest.DigestUtils;
public static String getUserToken(String account) {
    String md5 = "";
	md5 = DigestUtils.md5Hex(account+"0.0.0.0"+new Date());
	System.out.println(md5);
	return md5;
}

```

## 获取request所有参数，和Enumeration循环

```

 public Map<String, String> getRepParmas(HttpServletRequest request){
    	Enumeration<String>  params = 	request.getParameterNames();
	  	Map<String, String> reqparams = new HashMap<String, String>();
	  	while(params.hasMoreElements()){
	            String value = (String)params.nextElement();//调用nextElement方法获得元素
	            reqparams.put(value, request.getParameter(value));
	    }
	  	return reqparams;
}
```
#### 下载文件。。乱码等处理

```

import org.apache.commons.io.IOUtils;

      //修复IE下载 文件名乱码
    String userAgent = req.getHeader("user-agent").toLowerCase();  
      if (userAgent.contains("msie") || userAgent.contains("like gecko") ) {  
        // win10 ie edge 浏览器 和其他系统的ie  
        excelName = URLEncoder.encode(excelName, "UTF-8");  
    } else {  
        // fe  
      excelName = new String(excelName.getBytes("UTF-8"), "iso-8859-1");  
    }
      
    rsp.setContentType("Application/Octet-Stream");
    PopSalaryService service = FORP.SPRING_CONTEXT.getBean(PopSalaryService.class);
    String filename = req.getServletContext().getRealPath("/disk-file/excel-template/pre-modeltemplate.xls");
    byte[] data = service.getFileByteArray(filename);
    rsp.setHeader("Content-Disposition", "attachment; filename=\"" + excelName + "\"");
    IOUtils.write(data, rsp.getOutputStream());
	return null;

```

#### Map循环

```
    HashMap<String, Object> map = (HashMap<String, Object>)obj;
		Iterator<String> keys = map.keySet().iterator();
		while(keys.hasNext()){
			String k = keys.next();
			if(k.equals(key))
			{
				return map.get(k);
			}
		}
	

```
#### 24位编码
```
 /**
	  * 24位编码： 17日期+6随机数+"N"
	  * @return
	  */
		public static String getModelCode() {
			String ret = DateFormatUtils.format(new Date(), "yyyyMMddHHmmssSSS")+(int)((Math.random()*9+1)*100000)+"N";
			return ret;
		}

```

## 判断对象为空 判断对象不为空

```

/**
	 * 判断对象为空
	 * 
	 * @param obj
	 * @return
	 */
	public static boolean isEmpty(Object obj) {
		if (obj == null)
			return true;

		if (obj instanceof CharSequence)
			return ((CharSequence) obj).length() == 0;

		if (obj instanceof Collection)
			return ((Collection) obj).isEmpty();

		if (obj instanceof Map)
			return ((Map) obj).isEmpty();

		if (obj instanceof String)
			return "".equals(obj);

		if (obj instanceof Object[]) {
			Object[] object = (Object[]) obj;
			if (object.length == 0) {
				return true;
			}
			boolean empty = true;
			for (int i = 0; i < object.length; i++) {
				if (!isEmpty(object[i])) {
					empty = false;
					break;
				}
			}
			return empty;
		}
		return false;
	}

	/**
	 * 判断对象不为空
	 * 
	 * @param obj
	 * @return
	 */
	public static boolean isNotEmpty(Object obj) {
		return !isEmpty(obj);
	}
	
```
#### Arrays 工具

```
String[] arr ={"1","aaa2","3aaa","asds4"};
String arrString = Arrays.toString(arr);
System.out.println(arrString);

```
## JSON 相关


```
FastJson 
按顺序
Map<String, Object> itemMap = JSONObject.parseObject(exportFiled, LinkedHashMap.class);

```
## 日期相关

#### 计算两个日期相差天数  xx天 xx天xx时xx分xx秒

```
public  String getDatePoor(Date beginDate, Date endDate) {
		 
	    long nd = 1000 * 24 * 60 * 60;
	    long nh = 1000 * 60 * 60;
	    long nm = 1000 * 60;
	     long ns = 1000;
	    // 获得两个时间的毫秒时间差异
	    long diff = endDate.getTime() - beginDate.getTime();
	    // 计算差多少天
	    long day = diff / nd;
	    // 计算差多少小时
	    long hour = diff % nd / nh;
	    // 计算差多少分钟
	    long min = diff % nd % nh / nm;
	    // 计算差多少秒//输出结果
	     long sec = diff % nd % nh % nm / ns;
	    return day + "天" + hour + "小时" + min + "分" + sec + "秒";
	}
```
#### 

```
/**
	 * 计算两个日期之间相差的天数
	 * 
	 * @param smdate 较小的时间
	 * @param bdate 较大的时间
	 * @return 相差天数
	 * @throws ParseException
	 */
	public static int daysBetween(Date smdate, Date bdate) throws ParseException {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
		smdate = sdf.parse(sdf.format(smdate));
		bdate = sdf.parse(sdf.format(bdate));
		Calendar cal = Calendar.getInstance();
		cal.setTime(smdate);
		long time1 = cal.getTimeInMillis();
		cal.setTime(bdate);
		long time2 = cal.getTimeInMillis();
		long between_days = (time2 - time1) / (1000 * 3600 * 24);
		return Integer.parseInt(String.valueOf(between_days));
	}

```

####  string2Date

```

/**
	 * 字符串转日期
	 * 
	 * @param strDate 字符串日期
	 * @param pattern 日期格式
	 * @return
	 * @throws ParseException
	 */
	public static Date string2Date(String strDate, String pattern) throws ParseException {
		SimpleDateFormat sdf = new SimpleDateFormat(pattern);
		Date date = sdf.parse(strDate);
		return date;
	}
```
#### date2String

```
	/**
	 * 日期转字符串
	 * 
	 * @param date
	 * @return
	 * @throws ParseException
	 */

	public static String date2String(Date date, String pattern) throws ParseException {
		SimpleDateFormat formatter = new SimpleDateFormat(pattern);
		return formatter.format(date);
	}

```
#### getFirstDayOfMonth 获取指定年月的第一天

```
	/**
     * 获取指定年月的第一天
     * @param year
     * @param month
     * @return
     */
    public static String getFirstDayOfMonth(int year, int month) {     
        Calendar cal = Calendar.getInstance();   
        //设置年份
        cal.set(Calendar.YEAR, year);
        //设置月份 
        cal.set(Calendar.MONTH, month-1); 
        //获取某月最小天数
        int firstDay = cal.getMinimum(Calendar.DATE);
        //设置日历中月份的最小天数 
        cal.set(Calendar.DAY_OF_MONTH,firstDay);  
        //格式化日期
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.format(cal.getTime());  
    }
    public  String getLastDayOfMonth1(String yearmonth) { 
  	  int year = Integer.parseInt(yearmonth.split("-")[0]);
  	  int month =  Integer.parseInt(yearmonth.split("-")[1]);
  	  return getLastDayOfMonth1(year,month);
  	}
```
#### getLastDayOfMonth 获取指定年月的最后一天

```
	/**
     * 获取指定年月的最后一天
     * @param year
     * @param month
     * @return
     */
     public  String getLastDayOfMonth1(int year, int month) {     
         Calendar cal = Calendar.getInstance();     
         //设置年份  
         cal.set(Calendar.YEAR, year);  
         //设置月份  
         cal.set(Calendar.MONTH, month-1); 
         //获取某月最大天数
         int lastDay = cal.getActualMaximum(Calendar.DATE);
         //设置日历中月份的最大天数  
         cal.set(Calendar.DAY_OF_MONTH, lastDay);  
         //格式化日期
         SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");  
         return sdf.format(cal.getTime());
     }

```


## 接口方面

#### webservice

```

public static String postMethod(String url,String method,Object[] param){
           Service s = new  Service();
           String val =null;
           Call call;
			try {
				call = (Call) s.createCall();
			    call.setTargetEndpointAddress(url);
	            call.setOperation(method);
	            call.setTimeout(new Integer(5000));
	            val = (String)call.invoke(param);
	            System.out.println("method:"+ method+",param:"  + param+",return:"  + val);
			} catch (Exception e) {
				e.printStackTrace();
			}
		  	return val;
		}
	  public static String postMethod(String url,String method,Object[] param, List in){
          Service s = new  Service();
          String val =null;
          Call call;
			try {
				call = (Call) s.createCall();
			    call.setTargetEndpointAddress(url);
	            call.setOperation(method);
	            call.setTimeout(new Integer(5000));
	            for (Iterator iterator = in.iterator(); iterator.hasNext();) {
					String type = (String) iterator.next();
					call.addParameter(type, org.apache.axis.encoding.XMLType.XSD_STRING, javax.xml.rpc.ParameterMode.IN);
				}
	              val = (String)call.invoke(param);
	            System.out.println("method:"+ method+",param:"  + param+",return:"  + val);
			} catch (Exception e) {
				e.printStackTrace();
			}
		  	return val;
		}
```
#### rest 请求

```
	  /**
	   * rest 请求
	   * @param url
	   * @param param
	   * @return
	   */
	  public static String postMethod(String url,String method,String params){
          String val ="";
		try {
			 URL restServiceURL = new URL(url);
             HttpURLConnection httpConnection = (HttpURLConnection) restServiceURL.openConnection();
             httpConnection.setRequestMethod(method);
             httpConnection.setRequestProperty("Accept", "application/json");
             httpConnection.setDoOutput(true);     //需要输出
             httpConnection.setDoInput(true);      //需要输入
//             Iterator<Map.Entry<String, Object>> entries = params.entrySet().iterator(); 
//             while (entries.hasNext()) { 
//               Map.Entry<String, Object> entry = entries.next(); 
//               System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getKey()); 
//               httpConnection.setRequestProperty(entry.getKey(), entry.getKey());
//             }
             
             
             //建立输入流，向指向的URL传入参数
             DataOutputStream dos=new DataOutputStream(httpConnection.getOutputStream());
             dos.writeBytes(params);
             dos.flush();
             
             if (httpConnection.getResponseCode() != 200) {
                    throw new RuntimeException("HTTP GET Request Failed with Error code : "
                                  + httpConnection.getResponseCode());
             }
             BufferedReader responseBuffer = new BufferedReader(new InputStreamReader(
                    (httpConnection.getInputStream())));

             String output;
//             System.out.println("Output from Server:  \n");
             
             while ((output = responseBuffer.readLine()) != null) {
                    System.out.println(output);
                    val = val + output;
             }

             httpConnection.disconnect();
			 System.out.println("url:"+ url+",params:"  + params+",return:"  + val);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return val;
	}
	public static void main(String[] args) throws ConfigurationException {
		Object[] objects = {"taskid"};
		List<String> in = new ArrayList<String>();
		in.add("taskid");
		Map<String, Object> params = new HashMap<String, Object>();
			params.put("acc", "admin");
			params.put("pwd", "q7A1ArKxRM8=");
//				params.put("accessToken", "");
		String pa = "acc=admin&pwd=q7A1ArKxRM8=";
		String par = "accessToken=MpOfjtPXtYGGrQhBU3mHNH/TmQZNUoN8uNBzCd9fTGxPxVnCjxR2+m5p/DTWySzX";
		
//			postMethod("http://10.32.71.85:9080/KayangWebApis/KayangWebApi/Data/StartSession",
//					"POST",pa);
		postMethod("http://10.32.71.85:9080/KayangWebApis/KayangWebApi/Data/CloseSession",
				"POST",par);
		
//			postMethod("http://10.32.1.91:7007/OAUAcenter/services/OAOrgService",
//					"getOrgData",null);
//			postMethod("http://10.32.1.91:7007/OAUAcenter/services/OAProcessTaskService",
//					"endDb",objects);
//			getUserToken("admin");
	}

```
## 反射常用调用

```
  String methodName = new StringBuffer("get").append(key.substring(0, 1).toUpperCase())
	        	.append(key.substring(1)).toString();
	  Method method =  PrjProject.class.getMethod(methodName);
	  Object result = method.invoke(info);
```

## jdbc 数据操作

```
 String sql = " select count(1) as count from forp_userrole t where t.fk_userid = ? and t.fk_roleid = ? ";
 SqlRowSet rs = jdbc.queryForRowSet(sql, userId, roleid);
 if (rs.next()) {
    rs.getInt("count");
 }

```