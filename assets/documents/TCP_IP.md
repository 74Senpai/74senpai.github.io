## Các thành phần cần có để tạo được một ứng dụng TCP\IP đơn giản
> [!NOTE]
> Tài liệu chỉ để tham khảo

### 1. Thread
- Thread để tạo luồng chạy song song với luồng chính (main thread)
- Ứng dụng chính:
    + Giúp ứng dụng mượt mà, không bị treo
    + Xử lý nhiều tác vụ song song, không cần đợi tác vụ trước hoàn thành
    + Tách biệt tác vụ

- Khởi tạo & sử dụng:
```java
//Cách 1:
//Khởi tạo Class:
public class MyThread extends Thread {

    MyThread() {
    } // Hàm khởi tạo, có thể có hoặc không tùy trường hợp

    @Override
    public void run() {
    } // Logic chính của thread khi được chạy
}

//Sử dụng: 
public class Main {

    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start(); // Khởi chạy thread
    }
}

// Cách 2: 
// Khởi tạo, sử dụng bằng lambda
public class Main {

    public static void main(String[] args) {
        Thread myThread = new Thread(()->{
            // Logic chính của thread khi được chạy
        });
        myThread.start(); // Khởi chạy thread
    }
}
```

### 2. Socket
- Lý thuyết: 
    - Socket là một giao diện lập trình ứng dụng (`API-Application Programming Interface`)
    - Socket cho phép thiết lập các kênh giao tiếp mà hai đầu kênh được đánh dấu bởi hai cổng (`Port`)
    - Thông qua các cổng này một quá trình có thể nhận và gởi dữ liệu với các quá trình khác
    - Để thực hiện giao tiếp giữa các đối tượng, 1 trong số các đối tượng phải công bố số cổng (`Port`) của mình, đối tượng công bố số cổng để đối tượng khác kết nối vào gọi là máy chủ (`Server`) đối tượng kết nối vào gọi là khách (`Client`)
    - Ngoài ra các đối tượng phải biết địa chỉ IP (`IP Adress`) của nhau để định danh.

- Các thành phần bắt buộc của một ứng dụng mạng: 
    - `Socket`: Cần có để thiết lập các kênh giao tiếp.
    - `Port`: Cần có để 2 kênh `socket` tìm thấy nhau.
    - `IP Adress`: Cần có để 2 đối tượng tìm thấy, định danh nhau.

### 3. TCP
- TCP là cơ chế có kết nối, sử dụng kết nối ảo của 2 `Socket` để duy trì kết nối.
- Khung của ứng dụng TCP:
> Server: Lắng nghe, xử lý các yêu cầu của `Client`
```java
import java.net.*;
import java.io.*;

public class ServerTCP {

    private ServerSocket ss;
    private int port;

    // Hàm khởi tạo seversocket + lắng nghe client
    public void startServer(int port) {
        this.port = port;
        try {
            this.ss = new ServerSocket(this.port);
            new Thread(() -> { listenClient(); }).start();
        } catch (Exception e) {}
    }

    // Hàm đóng server
    public void closeServer() {
        try {
            if (this.ss != null) {
                this.ss.close();
            }
        } catch (Exception e) {}
    }

    // Hàm lắng nghe client socket
    private void listenClient() {
        while (!this.ss.isClosed()) {
            try {
                Socket s = ss.accept();
                new Thread(() -> handleClient(s)).start();
            } catch (Exception e) {}
        }
    }

    // Hàm xử lý I/O cho Client
    private void handleClient(Socket s) {
        try (DataInputStream in = new DataInputStream(
                new BufferedInputStream(s.getInputStream())); 
            DataOutputStream output = new DataOutputStream(s.getOutputStream()); ) {
            while (true) {
                String message = in.readUTF();
                String data = processClientRequest(message);
                output.writeUTF(data);
                output.flush();
            }
        } catch (Exception e) {}
    }

    // Hàm xử lý dữ liệu Client gửi lên, tính PTB2, ...
    private String processClientRequest(String data) {
        // Xử lý yêu cầu của người dùng
        return processedData;
    }

    // Main chạy server
    public static void main(String[] args) {
        new ServerTCP().startServer(3636);
    }
}
``` 

> Client: Kết nối tới `Server` gửi và nhận dữ liệu.
```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class ClientTCP {

    private Socket socket;
    private DataInputStream in;
    private DataOutputStream out;

    // Hàm khởi tạo kết nối
    public void startClient(String ip, int port) {
        try {
            this.socket = new Socket(ip, port);
            this.in = new DataInputStream(new BufferedInputStream(socket.getInputStream()));
            this.out = new DataOutputStream(socket.getOutputStream());

            // Luồng nhận dữ liệu từ Server
            new Thread(() -> { listenServer(); }).start();

            // Luồng gửi dữ liệu lên Server
            sendRequest();

        } catch (Exception e) {}
    }

    // Hàm đóng kết nối
    public void closeClient() {
        try {
            if (this.socket != null) {
                this.socket.close();
            }
        } catch (Exception e) {}
    }

    // Hàm lắng nghe dữ liệu từ Server
    private void listenServer() {
        try {
            while (true) {
                String response = in.readUTF();
                processServerResponse(response);
            }
        } catch (Exception e) {
            closeClient();
        }
    }

    // Hàm xử lý gửi yêu cầu
    private void sendRequest() {
        try {
            while (true) {
                String message = getInput();
                out.writeUTF(message);
                out.flush();
            }
        } catch (Exception e) {
            closeClient();
        }
    }

    // Hàm lấy input (từ bàn phím)
    private String getInput() {
        Scanner sc = new Scanner(System.in);
        return sc.nextLine();
    }

    // Hàm xử lý dữ liệu nhận được từ Server (hiển thị UI, xử lý logic...)
    private void processServerResponse(String data) {
        // Logic xử lý phản hồi từ Server
    }

    // Main chạy client
    public static void main(String[] args) {
        new ClientTCP().startClient("localhost", 3636);
    }
}
```

> [!TIP]
> Có thể tách các hàm xử lý dữ liệu, ... ra class riêng để không phình to class chính
