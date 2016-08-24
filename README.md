# android-serialport-api


##修复google 提供的串口工具android-serialport-api丢失字节bug。

###由于在issues多人提出丢失字节的bug，却没人修改，所以程梦真通过如下方式修复该bug，其真实的bug原因是由于关闭Activity的时候输入流的read方法被阻塞，以至于关闭Activity，但GC不能释放，导致下一次接收导数据时，数据先给了没有被释放的Activity，然后之后收到的数据才给新创建的Activity。


**1.由于测试App需要测试数据能否通过串口传输，所以使用了google的串口测试app。**

 - [app地址](https://code.google.com/archive/p/android-serialport-api/downloads)
 
 
**2.后由测试[刘涛](https://github.com/TonySudo)测试出 重复开关接收串口数据的activity，会导致数据丢失**

**3.和[曾剑锋](https://github.com/AplexOS)讨论可是能线程中inputstream.read()这个阻塞方法导致线程没走完，线程非静态对activity持有隐示应用，造成acvitity泄露，一旦有数据进入，就会被上一个activity线程中的inputstream.read()给吃掉一个数据**

**4.使用MAT进行内存分析，发现多次重复打开ConsoleActivity的对象在GC中并未回收**

**5.所以在inputstream.read()之前加个if (mInputStream.available() > 0) 的判断**

        1.  在SerialPortActivity中ReadThread添加

        if (mInputStream == null) return;
    					if (mInputStream.available() > 0) {
    						size = mInputStream.read(buffer);
    						if (size > 0) {
    							onDataReceived(buffer, size);
    						}
    					}
    		SystemClock.sleep(100);
  		
        2.  ReadThread的完整代码为
        
                private class ReadThread extends Thread {
              		@Override
              		public void run() {
              			super.run();
              			while(!isInterrupted()) {
              				int size;
              				try {
              					byte[] buffer = new byte[64];
              					if (mInputStream == null) return;
              					if (mInputStream.available() > 0) {
              						size = mInputStream.read(buffer);
              						if (size > 0) {
              							onDataReceived(buffer, size);
              						}
              					}
              					SystemClock.sleep(100);
              				} catch (IOException e) {
              					e.printStackTrace();
              					return;
              				}
              			}
              		}
              	}

**6.修改后用内存测试工具，解决了泄露的问题，每一次activity都被释放了。**
 
[修改后项目地址](https://github.com/chengcnaplex/android-serialport-api)

