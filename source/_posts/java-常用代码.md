---
title: java 常用代码
date: 2018-09-20 03:24:23
tags: 
categories: java
description: "java常用代码汇总，读取配置文件，日期，接口调用，是否为空，"
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
	public Map<String, String> getReqParams(HttpServletRequest request){
		Enumeration<String>  params = 	request.getParameterNames();
	  	Map<String, String> reqparams = new HashMap<String, String>();
	  	while(params.hasMoreElements()){
	            String value = (String)params.nextElement();//调用nextElement方法获得元素
	            reqparams.put(value, request.getParameter(value));
	    }
	  	return reqparams;
	}

	/*
	*
	*通过request获取参数
	*/
	public StringBuffer getParams(HttpServletRequest request){
        Enumeration paramMap= request.getParameterNames();
        StringBuffer sb = new StringBuffer();
        while(paramMap.hasMoreElements()){
            String key = (String) paramMap.nextElement();
            sb.append(key).append("=").append(request.getParameter(key).toString()).append("&");
        }
        return sb;
    }
```

####  get 获取文件内容并下载，变为byte[]，out输出  
```

  public static InputStream getInputStreamByUrl(String strUrl){
        HttpURLConnection conn = null;
        try {
            URL url = new URL(strUrl);
            conn = (HttpURLConnection)url.openConnection();
            conn.setRequestMethod("GET");
            conn.setConnectTimeout(20 * 1000);
            final ByteArrayOutputStream output = new ByteArrayOutputStream();
            IOUtils.copy(conn.getInputStream(),output);
            return  new ByteArrayInputStream(output.toByteArray());
        } catch (Exception e) {
            try{
                if (conn != null) {
                    conn.disconnect();
                }
            }catch (Exception e1){
            }
        }
        return null;
    }
    public  byte[] readBytes(InputStream in) throws IOException {  
        BufferedInputStream bufin = new BufferedInputStream(in);  
        int buffSize = 1024;  
        ByteArrayOutputStream out = new ByteArrayOutputStream(buffSize);  
  
        // System.out.println("Available bytes:" + in.available());  
  
        byte[] temp = new byte[buffSize];  
        int size = 0;  
        while ((size = bufin.read(temp)) != -1) {  
            out.write(temp, 0, size);  
        }  
        bufin.close();  
        in.close();  
        byte[] content = out.toByteArray();  
        out.close();  
        return content;  
    }

```
然后输出到页面,commons-io-2.6.jar

```
import org.apache.commons.io.IOUtils;

	response.setCharacterEncoding("UTF-8");
	response.setContentType("application/octet-stream");
	response.setHeader("charset", "utf-8");
	
	Tools tools = new Tools();
	StringBuffer sb  = tools.getParams(request);
	String fileId = request.getParameter("fileId");
	String fileName = request.getParameter("fileName");
	String url = "";
	
	fileName = new String(URLDecoder.decode(fileName, "UTF-8").getBytes("UTF-8"), "ISO8859-1");
	response.addHeader("Content-Disposition", "attachment;filename=\"" + fileName + "\";filename*=UTF-8''" + fileName);

	InputStream inp = tools.getInputStreamByUrl(url);
	byte[] bytes = tools.readBytes(inp);
	IOUtils.write(bytes, response.getOutputStream());
	response.flushBuffer();
	out.clear();
	out = pageContext.pushBody();
```

#### 下载文件。乱码等处理
commons-io-2.6.jar
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
#### 存数据库乱码变成问号,全角问题

```

	    byte[] space = new byte[]{(byte) 0xc2,(byte) 0xa0};
        String UTFSpace =new String( space,"UTF-8" );
        String  result=attachment.getOriginalFilename().replaceAll(UTFSpace, " ");

```
#### Map循环,获取request参数

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

#### 生成密码并验证，使用了正则，

正则， 包含大小写字母和数字，可以包含特殊字符 ，10位


```
	public static void main(String[] args) {
		UserService service = new UserService();
//		String pattern = "/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[\S]{8,16}$/";
//	    boolean isMatch = Pattern.matches(pattern, pass);
		Pattern pattern = Pattern.compile("^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)[\\S]{8,16}$"); 
	    String pass = service.createPassWord(10);
	    Matcher matcher = pattern.matcher(pass); 
	    boolean isMatch =   matcher.matches();
//		System.out.println(pass+":"+isMatch+":"+service.getDigPwd("MuWajayC1G"));
	}

	//生成密码
	private String generatePwd(){
        String pass = createPassWord(10);
		Pattern pattern = Pattern.compile("^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)[\\S]{8,16}$"); 
		Matcher matcher = pattern.matcher(pass); 
	    boolean isMatch =   matcher.matches();

	      if(isMatch){
				return pass;
		  }else{
			  return generatePwd();
		  }
	}
	// 加密Password
	private String getDigPwd(String pwd){
		return DigestUtils.md5Hex(FORP.MD5_SALT_PREFIX + pwd);
	}
	private String createPassWord(int len){
	    int random = this.createRandomInt();
	    return this.createPassWord(random, len);
	}
	private String createPassWord(int random,int len){
	    Random rd = new Random(random);
	    final int maxNum = 62;
	    StringBuffer sb = new StringBuffer();
	    int rdGet;//取得随机数
	    char[] str = { 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k',
	        'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w',
	        'x', 'y', 'z', 'A','B','C','D','E','F','G','H','I','J','K',
	        'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W',
	        'X', 'Y' ,'Z', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' };
	    int count=0;
	    while(count < len){
	      rdGet = Math.abs(rd.nextInt(maxNum));//生成的数最大为62-1
	      if (rdGet >= 0 && rdGet < str.length) {
	        sb.append(str[rdGet]);
	        count ++;
	      }
	    }
	    return sb.toString();
	  }
	  private int createRandomInt(){
	    //得到0.0到1.0之间的数字，并扩大100000倍
	    double temp = Math.random()*100000;
	    //如果数据等于100000，则减少1
	    if(temp>=100000){
	      temp = 99999;
	    }
	    int tempint = (int)Math.ceil(temp);
	    return tempint;
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
#### rest 请求 get and post use HttpURLConnection

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
	
	
	 /** 
     * 发起http请求获取返回结果 
     * @param req_url 请求地址 
     * @return 
     */ 
    public  String getMethod(String req_url) {
        StringBuffer buffer = new StringBuffer();  
        try {  
            URL url = new URL(req_url);  
            HttpURLConnection httpUrlConn = (HttpURLConnection) url.openConnection();  

            httpUrlConn.setDoOutput(false);  
            httpUrlConn.setDoInput(true);  
            httpUrlConn.setUseCaches(false);  

            httpUrlConn.setRequestMethod("GET");  
            httpUrlConn.connect();  

            // 将返回的输入流转换成字符串  
            InputStream inputStream = httpUrlConn.getInputStream();  
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");  
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);  

            String str = null;  
            while ((str = bufferedReader.readLine()) != null) {  
                buffer.append(str);  
            }  
            bufferedReader.close();  
            inputStreamReader.close();  
            // 释放资源  
            inputStream.close();  
            inputStream = null;  
            httpUrlConn.disconnect();  

        } catch (Exception e) {  
            System.out.println(e.getStackTrace());  
        }  
        return buffer.toString();  
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
 String sql = " select count(1) as count from demo_userrole t where t.fk_userid = ? and t.fk_roleid = ? ";
 SqlRowSet rs = jdbc.queryForRowSet(sql, userId, roleid);
 if (rs.next()) {
    rs.getInt("count");
 }

```

## log Linux 权限问题


通过继承RollingFileAppender或者DailyRollingFileAppender来实现

```
log4j.properties配置

#输出到文件   
log4j.appender.fileInfo = com.core.log4jconfig.Mylog4jWriter
log4j.appender.fileInfo.Threshold = DEBUG   
log4j.appender.fileInfo.layout = org.apache.log4j.PatternLayout   
log4j.appender.fileInfo.layout.ConversionPattern = %d{yyyy-MM-dd HH\:mm\:ss} %p %c %x - %m%n  
log4j.appender.fileInfo.Append = TRUE   
log4j.appender.fileInfo.File = /data/my/logs/my.log   
log4j.appender.fileInfo.File='.'yyyy-MM-dd  

```

com.core.log4jconfig.Mylog4jWriter.java代码

```
public class Mylog4jWriter extends DailyRollingFileAppender{  
      
    @Override  
    public synchronized void setFile(String fileName, boolean append,  
            boolean bufferedIO, int bufferSize) throws IOException {  
        super.setFile(fileName, append, bufferedIO, bufferSize);  
        File f = new File(fileName);  
        Set<PosixFilePermission> set = new HashSet<PosixFilePermission>();  
        set.add(PosixFilePermission.OWNER_READ);  
        set.add(PosixFilePermission.OTHERS_WRITE);  
        set.add(PosixFilePermission.GROUP_READ);  
        set.add(PosixFilePermission.OTHERS_READ);  
        if(f.exists()){  
            Files.setPosixFilePermissions(f.toPath(), set);  
        }  
    }  
  
} 

```
启动项目即可
生成的日志文件读写权限为rw-r--r--

所有用户都有日志文件的读权限

#### 修改 Sequence

```
public void updateSequence() {
  String sql = "select SEQUENCE_NAME from ALL_SEQUENCES WHERE SEQUENCE_OWNER='database'";
  List<xxxx> list = findByList(sql, xxxx.class);
  String updateSql1 = "";
  String updateSql2 = "";
  String selectSql = "";
  for (xxxx p:list) {
      updateSql1 = "alter sequence "+p.getSequence_name()+" increment by 1000000";
      jdbc.update(updateSql1);
      selectSql = "select "+p.getSequence_name()+".nextval from dual";
      List<xxxx> byList = findByList(selectSql, xxxx.class);
      updateSql2 = "alter sequence "+p.getSequence_name()+" increment by 1";
      jdbc.update(updateSql2);
  }
}
```