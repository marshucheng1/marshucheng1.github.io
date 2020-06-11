---
layout: post
title: FTPClient彻底解决乱码问题
date: 2017-03-13 15:31:22
categories: java
tags: javaoop
author: MarsHu
---

* content
{:toc}

### java-FTP工具 ###
实际开发过程中，我们可能会有面向FTP的业务需求，这个时候，我们就会考虑如何与FTP服务器建立连接，如何

读取FTP服务器上文件内容，如何上传文件至FTP服务器等。apache为我们提供了一套解决方案。我们可以使用

commons-net包下提供的相关内容实现业务需求。maven导包如下：

	<dependency>
	  <groupId>commons-net</groupId>
	  <artifactId>commons-net</artifactId>
	  <version>3.1</version>
	</dependency>






### FTPClient乱码问题 ###
在实际开发过程中，大家或多或少都会遇到乱码问题，可能因为乱码进入不了指定的FTP目录，也可能因为乱码

不能正确读取到FTP文件内容。因为，国内主流使用的编码只有2种，GBK和UTF-8，这里我们可以编写一个FTP工具类。

具体代码如下：

	import org.apache.commons.net.ftp.FTPClient;
	import org.apache.commons.net.ftp.FTPFile;
	import org.apache.commons.net.ftp.FTPReply;
	
	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.InputStreamReader;
	import java.text.SimpleDateFormat;
	
	public class FTPUtil {
	
	    //检查ftp服务器是否是utf8编码
	    private static boolean is_utf8;
	
	    public static FTPClient getFTPClient(String ftpHost, Integer ftpPort, String ftpUserName, String ftpPassword) {
	        FTPClient ftpClient = null;
	        try {
	            ftpClient = new FTPClient();
	            ftpClient.setConnectTimeout(60000);
	            if (ftpPort != null) {
	                ftpClient.connect(ftpHost, ftpPort);
	            } else {
	                ftpClient.connect(ftpHost);
	            }
	            if (FTPReply.isPositiveCompletion(ftpClient.getReplyCode())) {
	                if (ftpClient.login(ftpUserName, ftpPassword)) {
	                    //判断FTP服务器是UTF8编码还是GBK编码
	                    if (FTPReply.isPositiveCompletion(ftpClient.sendCommand(
	                            "OPTS UTF8", "ON"))) {
	                        is_utf8 = true;
	                    }
	                    ftpClient.enterLocalPassiveMode();
	                    ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
	                    ftpClient.enterLocalPassiveMode();
	                }
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	        return ftpClient;
	    }
	
	    public static void getFTPFile(FTPClient ftpClient, String path, String fileName) {
	        try {
	            boolean check_work;
	            if (is_utf8) {
	                check_work = ftpClient.changeWorkingDirectory(new String(path.getBytes("UTF-8"), "iso-8859-1"));
	            } else {
	                check_work = ftpClient.changeWorkingDirectory(new String(path.getBytes("GBK"), "iso-8859-1"));
	            }
	            if (check_work) {
	                FTPFile[] files = ftpClient.listFiles();
	                if (files.length > 0) {
	                    String ftpFileName;
	                    String lastModifyTime;
	                    InputStream in;
	                    BufferedReader br;
	                    for (FTPFile ftpFile : files) {
	                        //获取FTP服务器文件名
	                        if (is_utf8) {
	                            ftpFileName = new String(ftpFile.getName().getBytes("iso-8859-1"), "UTF-8");
	                        } else {
	                            ftpFileName = new String(ftpFile.getName().getBytes("iso-8859-1"), "GBK");
	                        }
	                        //获取FTP服务器文件最后修改时间
	                        if (is_utf8) { //因为我们是+8区,utf8编码文件获取时间要加8小时
	                            lastModifyTime = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss").format(ftpFile.getTimestamp().getTimeInMillis() + ftpFile.getTimestamp().getTimeZone().getOffset(0));
	                        } else {
	                            lastModifyTime = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss").format(ftpFile.getTimestamp().getTime());
	                        }
	                    }
	                    //获取指定文件名的输入流
	                    if (is_utf8) {
	                        in = ftpClient.retrieveFileStream(new String(fileName.getBytes("UTF-8"), "iso-8859-1"));
	                    } else {
	                        in = ftpClient.retrieveFileStream(new String(fileName.getBytes("GBK"), "iso-8859-1"));
	                    }
	                    //解析文件内容
	                    if (in != null) {
							//这里也会有编码问题出现的可能--因为读取内容可能包含中文
	                        br = new BufferedReader(new InputStreamReader(in, "gb2312"));
	                        String data;
	                        while ((data = br.readLine()) != null) {
	                            System.out.println(data);
	                        }
	                    }
	                }
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            is_utf8 = false;
	            disConnection(ftpClient);
	        }
	    }
	
	    public static void disConnection(FTPClient ftpClient) {
	        try {
	            if (ftpClient.isConnected()) {
	                ftpClient.disconnect();
	            }
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	
	}

用了一个开关来控制是`UTF-8`编码还是`GBK`编码。大体上就是上面所示代码了。