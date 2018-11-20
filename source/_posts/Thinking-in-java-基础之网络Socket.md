---
title: Thinkin in java 基础之网络Socket
tags:
  - Socket
categories: java
description: "TCP/UDP 其他"
date: 2016-09-24 07:01:20
---

#### TCP

Server:

```
package Socket;
import java.net.*;
import java.io.*;
public  class TCPServer {
	public static void main(String[] args) throws Exception{
		ServerSocket ss=new ServerSocket(6666);
			while (true) {
				Socket s=ss.accept();
				System.out.println("hello word!");
				DataInputStream  Dim=new DataInputStream(s.getInputStream());
				System.out.println(Dim.readUTF());
				s.close();
			}
	}

}
```
TCPClient

```
package Socket;

import java.io.DataOutputStream;
import java.io.OutputStream;
import java.net.Socket;

public class TCPClient {
	public TCPClient() {
	}
	public static void main(String[] args) throws Exception{
		Socket s =new Socket("127.0.0.1",6666);
		OutputStream os=s.getOutputStream();
		DataOutputStream dos=new DataOutputStream(os);
		dos.writeUTF("Hello Server!");
		dos.flush();
		dos.close();
		s.close();
	}

}

```
#### UDP

UDPServer

```
package Socket;
import java.net.*;
import java.io.*;
public  class UDPServer {
	public static void main(String[] args) throws Exception {
		byte[] buf=new byte[1024];
		DatagramPacket dp=new DatagramPacket(buf,buf.length);
		DatagramSocket ds=new DatagramSocket(6666);
		while (true) {
			ds.receive(dp);
			System.out.println("IP:"+ds.getInetAddress());
			System.out.println("Port:"+ds.getPort());
		//System.out.println(new String(buf,0,dp.getLength()));
			ByteArrayInputStream bais=new ByteArrayInputStream(buf);
			DataInputStream dis=new DataInputStream(bais);
			System.out.println(dis.readLong());
		}
	}
}
```
UDPClient

```
package Socket;
import java.net.*;
import java.io.*;
public  class UDPClient {
	public static void main(String[] args)throws Exception {
		long i=10000L;
		ByteArrayOutputStream baos=new ByteArrayOutputStream();
		DataOutputStream dos=new DataOutputStream(baos);
		dos.writeLong(i);
		//byte[] buf=(new String("hello")).getBytes();
		byte[] buf=baos.toByteArray();
		System.out.println(buf.length);
		DatagramPacket dp=new DatagramPacket(buf,buf.length,new InetSocketAddress("127.0.0.1",6666));
		DatagramSocket ds=new DatagramSocket(8888);
		ds.send(dp);
		ds.close();
	}

}
```

