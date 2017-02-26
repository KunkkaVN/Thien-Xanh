** SERVER
---------

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package tictoeserver;

import java.io.*;
import java.net.*;

public class Tictoeserver {

    /**
     * @param args the command line arguments
     */
    private static ServerSocket servSock;
    private static final int PORT = 1996;
    public static void main(String[] args) throws IOException {
        try
        {
            servSock = new ServerSocket(PORT);
            System.out.println("Tic Toe Server is running");
            while (true)
            {
                Game game = new Game();
                Game.Player playerX = game.new Player(servSock.accept(),'X');
                Game.Player playerO = game.new Player(servSock.accept(), 'O');
                playerX.setOpponent(playerO);
                playerO.setOpponent(playerX);
                game.currentPlayer = playerX;
                playerX.start();
            }
        }
        catch (IOException e)
        {
            System.out.println("Error create socket!");
            System.exit(1);
        }
        finally {
            servSock.close();
        }
        
    }
    
}
```

class game

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package tictoeserver;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

/**
 *
 * @author HT
 */
public class Game {

    private Player[] board = {null, null, null,
        null, null, null,
        null, null, null};

    Player currentPlayer;
    
    public boolean hasWinner(){
        return (board[0]!= null && board[0]==board[1] && board[0]==board[2])
                || (board[3] != null && board[3]==board[4]&&board[3]==board[5])
                || (board[6] != null && board[6]==board[7] && board[6]==board[8])
                ||(board[0]!= null && board[0]==board[3] && board[0]==board[6])
                ||(board[1] !=null && board[1]==board[4] && board[1]==board[7])
                ||(board[2] != null && board[2]==board[5] && board[2]==board[8])
                ||(board[0] !=null && board[0]==board[4] && board[0]==board[8])
                ||(board[2] !=null && board[2]==board[4] && board[2]==board[6]);
    }
    
    public synchronized boolean legalMove(int location, Player player) {
            if (player == currentPlayer && board[location] == null) {
           board[location] = currentPlayer;
                currentPlayer = currentPlayer.opponent;
                currentPlayer.otherPlayerMoved(location);
                return true;
            }
            return false;
        }
    public boolean boardFilledUp(){
        for (int i=0;i<board.length;i++){
            if (board[i] == null){
                return false;
            }
        }
        return true;
    }
    class Player extends Thread {

        char mark;

        Player opponent;
        Socket socket;
        BufferedReader input;
        PrintWriter output;

        public Player(Socket socket, char mark) throws IOException {
            this.socket = socket;
            this.mark = mark;
            try {
                input = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                output = new PrintWriter(socket.getOutputStream(), true);
                output.println("WELCOME " + mark);
                output.println("MESSAGE: waiting connect");
            } catch (IOException e) {
                System.out.println("Player died: " + e);
            }
        }

        public void setOpponent(Player opponent) {
            this.opponent = opponent;
        }

        public void otherPlayerMoved(int location) {
            output.println("doi thu di chuyen " + location);
            output.println(hasWinner() ? "DEFEAT" : boardFilledUp() ? "TIE" : "");
        }

        public void run() {
            try {
                output.println("MESSAGE: tất cả Player đã kết nối ");
                if (mark == 'x') {
                    output.println("MESSAGE: tiếp theo");
                }
                while (true) {
                    String command = input.readLine();
                    if (command.startsWith("MOVE")) {
                        int location = Integer.parseInt(command.substring(5));
                        if (legalMove(location, this)) {
                            output.println("VALID_MOVE");
                            output.println(hasWinner() ? "VICTORY" : boardFilledUp() ? "TIE" : "");
                        } else {
                            output.println("MESSAGE: ?");
                        }
                    } else if (command.startsWith("QUIT")) {
                        return;
                    }
                }
            } catch (IOException e) {
                System.out.println("Player died: " + e);
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {

                }
            }
        }        
    }
}
```

*2. Client: 2 th moi choi dc
----------------------------

```sh
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package tictactoeclient;

import javax.swing.*;
import java.awt.*;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.io.*;
import java.net.*;
/**
 *
 * @author HT
 */
public class TicTacToeClient {
    private JFrame frame = new JFrame("Tic Tac Toe");
    private JLabel messageLabel = new JLabel("");
    private ImageIcon icon;
    private ImageIcon opponentIcon;
    
    private Square[] board = new Square[9];
    private Square currentSquare;
    
    private static int PORT = 1996;
    private Socket socket;
    private BufferedReader in;
    private PrintWriter out;
    
    public TicTacToeClient(String serverAddress) throws IOException{
        socket = new Socket(serverAddress,PORT);
        in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        out = new PrintWriter(socket.getOutputStream(), true);
        
        messageLabel.setBackground(Color.lightGray);
        frame.getContentPane().add(messageLabel,"South");
        
        JPanel boardPanel = new JPanel();
        boardPanel.setBackground(Color.black);
        boardPanel.setLayout(new GridLayout(3, 3, 2, 2));
        for (int i =0; i < board.length;i++){
            final int j =i;
            board[i]=new Square();
            board[i].addMouseListener(new MouseAdapter(){
                public void mousePressed(MouseEvent e) {
                    currentSquare = board[j];
                    out.println("MOVE "+j);  
                }
            });
            boardPanel.add(board[i]);
        }
        frame.getContentPane().add(boardPanel,"Center");
    }
    public void Play() throws IOException{
        String response;
        try{
            response = in.readLine();
            if (response.startsWith("WELCOME")){
                char mark = response.charAt(8);
                icon = new ImageIcon(mark == 'X' ? " x.gif" : "o.gif");
                opponentIcon = new ImageIcon(mark == 'x' ? "o.gif" : "x.gif");
                frame.setTitle("Tic Tac Toe - Player "+mark);
            }
            while (true) {
                response = in.readLine();
                if (response.startsWith("VALID_MOVE")){
                    messageLabel.setText("Valid move, wait");
                    currentSquare.setIcon(icon);
                    currentSquare.repaint();
                }
                else if (response.startsWith("doi thu di chuyen")) {
                    int loc = Integer.parseInt(response.substring(15));
                    board[loc].setIcon(opponentIcon);
                    board[loc].repaint();
                    messageLabel.setText("Opponent moved");
                }
                else if (response.startsWith("VICTORY")){
                    messageLabel.setText("You Win!");
                    break;
                }
                else if (response.startsWith("DEFEAT")){
                    messageLabel.setText("You Lose!");
                    break;
                }
                else if (response.startsWith("TIE")){
                    messageLabel.setText("You tied!");
                    break;
                }
                else if (response.startsWith("MESSAGE")){
                    messageLabel.setText(response.substring(8));
                }
            }
            out.println("QUIT");
        }
        finally{
            socket.close();
        }
    }
    private boolean WantToPlayAgain(){
        int response = JOptionPane.showConfirmDialog(frame,"Want to lay again?", "OK?",JOptionPane.YES_NO_OPTION);
        frame.dispose();
        return response == JOptionPane.YES_OPTION;
    }
    static class Square extends JPanel{
        JLabel label = new JLabel((Icon)null);
        public Square(){
            setBackground(Color.white);
            add(label);
        }
        public void setIcon(Icon icon){
            label.setIcon(icon);
        }
    }
    public static void main(String[] args) throws IOException {
        while(true){
            String serverAddress = (args.length == 0) ? "localhost":args[1];
            TicTacToeClient client = new TicTacToeClient(serverAddress);
            client.frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            client.frame.setSize(240,160);
            client.frame.setVisible(true);
            client.frame.setResizable(false);
            client.Play();
            if(!client.WantToPlayAgain()){
                break;
            }
        }
    }

}
```
