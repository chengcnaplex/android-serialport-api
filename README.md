# android-serialport-api


##修复google 提供的串口工具android-serialport-api丢失字节bug。由于在issues多人提出丢失字节的bug，却没人修改，所以作者做出如下修复。

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
