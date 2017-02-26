*1. Server:
-----------

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package server;

import java.io.*;
import java.net.*;
/**
 *
 * @author HT
 */
public class Server {

    /**
     * @param args the command line arguments
     */
    private static ServerSocket servSock;
    private static final int PORT = 1999;
    public static void main(String[] args) throws IOException {
        // TODO code application logic here
        
        System.out.println("Openning port \n");        
        try 
        {
            servSock = new ServerSocket(PORT);
        }
        catch (IOException e)
        {
            System.out.println("Error create socket!");
            System.exit(1);
        }
        do {
            Socket link=null;
            try{
            System.out.println("San san ket noi!");
            link = servSock.accept();
            System.out.println("Da ket noi!" + link.getLocalSocketAddress());
            BufferedReader in = new BufferedReader(new InputStreamReader(link.getInputStream()));
            PrintWriter out = new PrintWriter(link.getOutputStream(),true);
            BufferedReader reader =new BufferedReader(new InputStreamReader(System.in));            
            String msg = in.readLine();
            String msgOut;
            while (!msg.equals("BYE"))
            {
                System.out.println("Message received.");
                System.out.print("Enter message: ");
                msgOut= reader.readLine();
                out.println(msgOut);
                msg = in.readLine();
                System.out.println("CLIENT> " + msg);
            }
        }
        catch(Exception e) {
            System.out.println("loi io"+e.getMessage());
        }
        
        finally{
            try {
                link.close();
            }
            catch (IOException e){
                e.printStackTrace();
            }
        }
        }while (true);
    }
}
```

*2. Client:
-----------

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package client;

import java.io.*;
import java.net.*;
/**
 *
 * @author HT
 */
public class Client {

    /**
     * @param args the command line arguments
     */
    private static InetAddress host;
    private static final int port = 1999;
    public static void main(String[] args) throws IOException {
        // TODO code application logic here
        System.out.println("Dang ket noi\n");
        try{
            host = InetAddress.getLocalHost();            
        }
        catch(UnknownHostException e){
            System.out.println("Host not found!");
            System.exit(1);
        }
        finally{
            Socket sock = null;
            try {
                sock = new Socket("localhost",port);
                System.out.println("Da ket noi toi host:");
                BufferedReader in = new BufferedReader(new InputStreamReader (sock.getInputStream()));
                PrintWriter out = new PrintWriter(sock.getOutputStream(), true);
                BufferedReader reader =new BufferedReader(new InputStreamReader(System.in));
                String msgOut, msgIn;
                do
                {
                    System.out.print("Enter message: ");
                    msgOut = reader.readLine();
                    out.println(msgOut);
                    msgIn = in.readLine();
                    System.out.println("SERVER> " + msgIn);
                }while (!msgOut.equals("BYE"));
            }
            catch(IOException e)
            {
                e.printStackTrace();
            }
            finally{
                sock.close();
            }
        }
        
    }
}
```
