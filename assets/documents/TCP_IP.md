# TCP\IP đơn giản
> [!NOTE]
> Đây là tài liệu tham khảo cách xây dựng một ứng dụng TCP\IP đơn giản.
> 

## Các thành phần cần có để tạo được một ứng dụng TCP\IP đơn giản

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

### 3. Giao thức
*Các giao thức mạng để xây dựng ứng dụng, tùy vào nhu cầu mà mỗi ứng dụng sẽ cần các giao thức khác nhau*
#### 3.1 TCP
- TCP là cơ chế có kết nối, sử dụng kết nối ảo của 2 `Socket` để duy trì kết nối.
- Khung của ứng dụng TCP:
    + Flow: Server start (port) -> Client start (IP server, port) -> Server acept -> Handle

> Server: Lắng nghe, xử lý các yêu cầu của `Client`
```java
import java.net.*;
import java.io.*;

public abstract class ServerTCP {

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
            while (!s.isClosed()) {
                String message = in.readUTF();
                String data = processClientRequest(message);
                output.writeUTF(data);
                output.flush();
            }
        } catch (Exception e) {}
    }

    // Hàm xử lý dữ liệu Client gửi lên, tính PTB2, ...
    protected abstract String processClientRequest(String data);

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
            while (!this.socket.isClosed()) {
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
            while (!this.socket.isClosed()) {
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

#### 3.2 UDP
- UDP là cơ chế không kết nối, bên gửi chỉ gửi dữ liệu đến IP + Port, không thiết lập hay duy trì trạng thái kết nối với bên nhận.

- Khung:
    + Flow:
        - Server start (port) → Receive datagram → Handle
        - Client start → Send datagram (IP Server, port)

> Server: Khởi tạo socket, gán cổng nhận/gửi dữ liệu
```java
import java.net.*;

public abstract class ServerUDP {

    private DatagramSocket socket;
    private boolean running = false;

    public void startServer(int port) {
        try {
            socket = new DatagramSocket(port);
            running = true;
            new Thread(()->{this.listen();}).start();
        } catch (Exception ignored) {}
    }

    public void closeServer() {
        running = false;
        if (socket != null && !socket.isClosed()) {
            socket.close();
        }
    }

    // Lắng nghe gói tin gửi tới
    private void listen() {
        while (running) {
            try {
                byte[] buf = new byte[1024];
                DatagramPacket packet =
                        new DatagramPacket(buf, buf.length);

                socket.receive(packet);

                // Mỗi request một Thread để tăng tốc xử lý
                new Thread(() -> handlePacket(packet)).start();

            } catch (Exception ignored) {}
        }
    }

    // Xử lý một gói tin
    private void handlePacket(DatagramPacket packet) {
        try {
            String request = new String(
                    packet.getData(), 0, packet.getLength()
            );

            String response = processRequest(request);
            byte[] out = response.getBytes();

            DatagramPacket reply = new DatagramPacket(
                    out,
                    out.length,
                    packet.getAddress(),
                    packet.getPort()
            );

            socket.send(reply);

        } catch (Exception ignored) {}
    }

    // Hàm chưa logic xử lý
    protected abstract String processRequest(String data);
}

```

> Client: Khởi tạo, gửi/nhận dữ liệu
```java
import java.net.*;
import java.util.Scanner;

public class ClientUDP {
    private DatagramSocket socket;
    private InetAddress address;
    private int port;

    public void startSocket(String ip, int port) {
        try {
            this.port = port;
            this.socket = new DatagramSocket();
            this.address = InetAddress.getByName(ip);
        } catch (Exception ignored) {}
    }

    public void handlePacket() {
        try {
            while (!this.socket.isClosed()) {

                // Gửi dữ liệu
                byte[] outBuf = getInput().getBytes();
                DatagramPacket request =
                        new DatagramPacket(outBuf, outBuf.length, address, port);
                socket.send(request);

                // Nhận phản hồi
                byte[] inBuf = new byte[1024];
                DatagramPacket reply =
                        new DatagramPacket(inBuf, inBuf.length);
                socket.receive(reply);

                String received = new String(
                        reply.getData(), 0, reply.getLength()
                );

                processResponse(received);
            }
        } catch (Exception ignored) {}
    }

    public void processResponse(String data) {
        // Xử lý dữ liệu trả về (in ra, parse, ...)
        System.out.println("Server: " + data);
    }

    public void close() {
        if (socket != null && !socket.isClosed()) {
            socket.close();
        }
    }

    private String getInput() {
        Scanner sc = new Scanner(System.in);
        return sc.nextLine();
    }
}

```

> [!NOTE]
> Để tránh README dài dòng, các Exception được bỏ qua tuy nhiên khi triển khai phải xử lý Exception tránh nuốt lỗi.

> [!TIP]
> Có thể tách các hàm xử lý dữ liệu, ... ra class riêng để không phình to class chính.

## Tổng kết
- Thread: Phục vụ việc xử lý song song, tăng tốc độ phản hồi.
    + Nguyên do: 
        - Các hàm có thuộc tính `blocking` (accept, receive, read,...) chặn luồng chạy.
        - Thời gian phản hồi của một yêu cầu phụ thuộc hoàn toàn vào thời gian xử lý của chính nó 
        cộng với thời gian phản hồi của tác vụ trước. Nếu một tác vụ có thời gian là 1 đi sau tác vụ có thời gian phản hồi là 100, thời gian phản hồi thực tế là 101.
- Socket: API cần thiết để giao tiếp mạng.
- Giao thức:
    - Các dữ liệu chung: Cổng socket (`Port`), Địa chỉ IP (`IP address`)
    - Các hàm/phương thức cần thiết ở `server`:
        - `start()`: Hàm mở kết nối socket.
        - `stop()`: Hàm đóng kết nối socket.
        - `listen()`: Hàm lắng nghe các yêu cầu, dữ liệu.
        - `handle()`: Hàm xử lý I/O cho các yêu cầu, dữ liệu.
        - `process()`: Hàm tiện ích, chứa logic xử lý yêu cầu, dữ liệu nhằm tách biệt vài trò.

    ***listen/handle/process có thể gộp chung thành một hàm nếu muốn***

    - Các hàm/phương thức cần thiết ở `client`:
        - `start()`: Hàm mở kết nối socket.
        - `stop()`: Hàm đóng kết nối socket.
        - `listen()`: Hàm lắng nghe phản hồi.
        - `send()`: Hàm gửi dữ liệu, yêu cầu.
        - `process()`: Hàm tiện ích, chứa logic xử lý dữ liệu trả về.
        - `getInput()`: Hàm tiện ích lấy dữ liệu để gửi đi.
        
    ***có thể gộp chung các hàm nếu muốn***
