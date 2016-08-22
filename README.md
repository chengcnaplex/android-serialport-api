# android-serialport-api


##修复google 提供的串口工具android-serialport-api丢失字节bug。

由于在issues多人提出丢失字节的bug，却没人修改，所以程梦真通过如下方式修复该bug，其真实的bug原因是由于关闭Activity的时候输入流的read方法被阻塞，以至于关闭Activity，但GC不能释放，导致下一次接收导数据时，数据先给了没有被释放的Activity，然后之后收到的数据才给新创建的Activity。

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
