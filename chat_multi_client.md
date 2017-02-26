*1. Server chat:
----------------

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package socket_server;

import java.io.*;
import java.net.*;
import java.util.HashSet;
/**
 *
 * @author HT
 */
public class SOCKET_SERVER {

    /**
     * @param args the command line arguments
     */
    private static ServerSocket servSock;
    private static final int PORT = 1996;
    private static HashSet<String> names = new HashSet<String>();
    private static HashSet<PrintWriter> writers = new HashSet<PrintWriter>();
    
    public static void main(String[] args) throws IOException {
        System.out.println("Server chat running.");
        servSock= new ServerSocket(PORT);
        try {
            while(true){
                new allClient(servSock.accept()).start();
            }
        }
        finally{
            servSock.close();
        }
    }
    
    private static class allClient extends Thread{
        private String name;
        private Socket socket;
        private BufferedReader in;
        private PrintWriter out;

        private allClient(Socket socket) {
            this.socket=socket;
        }
        
        public void run(){
            try{
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(),true);
                while(true){
                    out.println("dang ki ten");
                    name=in.readLine();
                    synchronized(names){
                        if (!names.contains(name)){
                            names.add(name);
                            break;
                        }
                    }
                }
                System.out.println("Tai khoan dang ket noi: "+name);
                out.println("thanh cong");
                writers.add(out);
                while(true){
                    String input = in.readLine();
                    for (PrintWriter writer : writers){
                        writer.println("MESSAGE from "+name+" > "+input);
                    }
                }
            }
            catch (IOException e){
                System.out.println(e);
            }
            finally {
                if (name != null){
                    names.remove(out);
                }
                if (out !=null){
                    writers.remove(out);
                }
                try{
                    socket.close();
                }
                catch(IOException e){
                    
                }
            }
        }
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
package client_chat_muti;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.*;
import java.net.Socket;
import javafx.scene.text.TextFlow;
import javax.swing.*;

/**
 *
 * @author HT
 */
public class Client_chat_muti {

    BufferedReader in;
    PrintWriter out;
    
    JFrame frame = new JFrame("Chat");
    JTextField textfield= new JTextField(40);
    JTextArea textarea = new JTextArea(8,40);
    
    public Client_chat_muti(){
        textfield.setEditable(false);
        textarea.setEditable(false);
        frame.getContentPane().add(textfield,"North");
        frame.getContentPane().add(new JScrollPane(textarea),"Center");
        frame.pack();
        textfield.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                out.println(textfield.getText());
                textfield.setText("");
            }
        });
    }
    private String getname(){
        return JOptionPane.showInputDialog(frame,"Dat ten: ","Xac nhan",JOptionPane.PLAIN_MESSAGE);
    }
    private void run() throws IOException{
        Socket socket = new Socket("localhost",1996);
        in=new BufferedReader(new InputStreamReader(socket.getInputStream()));
        out = new PrintWriter(socket.getOutputStream(),true);
        while (true){
            String line=in.readLine();
            if (line.startsWith("dang ki ten")){
                out.println(getname());
            }
            else if (line.startsWith("thanh cong")){
                textfield.setEditable(true);
            }
            else if(line.startsWith("MESSAGE")){
                textarea.append(line.substring(10)+"\n");
            }
        }
    }
    public static void main(String[] args) throws IOException {
        Client_chat_muti client = new Client_chat_muti();
        client.frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        client.frame.setVisible(true);
        client.run();
    }
    
}
```
