### Các bước xây dựng RMI cơ bản

#### 1. Tạo các Interface cần thiết cho remote.
> Interface phải kế thừa `java.rmi.Remote`
> Các phương thức phải khai báo `throws RemoteException`

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface PTB2 extends Remote {

    String tinhPTB2(double a, double b, double c) throws RemoteException;
}
```

#### 2. Implement logic cho các Interface, đăng ký Remote
1. Từ Interface tiến hành Implement logic các phương thức.
2. Khởi tạo lớp Registry.
3. Khởi tạo, đăng ký Remote.
4. Tiến hành đợi triệu gọi từ xa.

***Step 1***
```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class RemotePTB2Server extends UnicastRemoteObject implements PTB2 {

    public RemotePTB2Server() throws RemoteException {
        super();
    }

    @Override
    public String tinhPTB2(double a, double b, double c) throws RemoteException {

        if (a == 0) {
            if (b == 0) {
                if (c == 0) {
                    return "Phuong trinh vo so nghiem"; 
                }else {
                    return "Phuong trinh vo nghiem";
                }
            }
            double x = -c / b;
            return "Phuong trinh co 1 nghiem x = " + x;
        }

        double delta = b * b - 4 * a * c;

        if (delta < 0) {
            return "Phuong trinh vo nghiem";
        } else if (delta == 0) {
            double x = -b / (2 * a);
            return "Phuong trinh co nghiem kep x = " + x;
        } else {
            double x1 = (-b + Math.sqrt(delta)) / (2 * a);
            double x2 = (-b - Math.sqrt(delta)) / (2 * a);
            return "Phuong trinh co 2 nghiem: x1 = " + x1 + ", x2 = " + x2;
        }
    }
}
```

***Step 2, 3, 4***
```java

import java.rmi.*;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class ServerMain {

    public static final String PTB2 = "server.PTB2";

    public static void main(String[] args) throws RemoteException {
        // Step 2
        final Registry registry = LocateRegistry.createRegistry(8080);

        //Step 3
        final RemotePTB2Server sv = new RemotePTB2Server();
        registry.rebind(PTB2, sv);

        //Step 4
        System.out.println("RMI Server dang chay...");
    }
}

```

#### 3. Tạo Client triệu gọi tới Remote
1. Tạo Registry, tìm cổng tới ServerRemote
2. Gọi đối tượng từ Registry.

```java
import java.rmi.AccessException;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class ClientMain {

    public static final String PTB2 = "server.PTB2";

    public static void main(String[] args) throws RemoteException, AccessException, NotBoundException {
        //Step 1
        final Registry registry = LocateRegistry.getRegistry("localhost",8080);

        //Step 2
        PTB2 ptb2 = (PTB2) registry.lookup(PTB2);
        String res = ptb2.tinhPTB2(3, 6, 3);
        System.out.println("Res: " + res);
    }
}
```
