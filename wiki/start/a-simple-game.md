import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class SimpleBrickGame extends JFrame implements Runnable, KeyListener {
    private int ballX = 150, ballY = 250;
    private int ballSize = 20;
    private int dx = 2, dy = 2;
    private int paddleX = 150, paddleY = 350;
    private int paddleWidth = 60, paddleHeight = 10;
    private int score = 0, lives = 3;
    private boolean[][] bricks = new boolean[3][7];
    private boolean gameRunning = true;

    public SimpleBrickGame() {
        setTitle("Simple Brick Game");
        setSize(400, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        addKeyListener(this);
        setVisible(true);
        
        // Initialize bricks
        for(int i=0; i<bricks.length; i++) {
            for(int j=0; j<bricks[0].length; j++) {
                bricks[i][j] = true;
            }
        }
        
        new Thread(this).start();
    }

    public void run() {
        while(gameRunning) {
            moveBall();
            repaint();
            try { Thread.sleep(10); } 
            catch (InterruptedException e) {}
        }
    }

    private void moveBall() {
        ballX += dx;
        ballY += dy;

        // Wall collisions
        if(ballX <= 0 || ballX >= getWidth()-ballSize) dx *= -1;
        if(ballY <= 0) dy *= -1;
        
        // Paddle collision
        if(ballY >= paddleY - ballSize && ballX >= paddleX && ballX <= paddleX + paddleWidth) {
            dy = -Math.abs(dy);
        }
        
        // Brick collisions
        for(int i=0; i<bricks.length; i++) {
            for(int j=0; j<bricks[0].length; j++) {
                if(bricks[i][j]) {
                    int brickX = j * 55 + 10;
                    int brickY = i * 25 + 50;
                    if(ballX >= brickX && ballX <= brickX + 50 &&
                       ballY >= brickY && ballY <= brickY + 20) {
                        bricks[i][j] = false;
                        dy *= -1;
                        score += 10;
                    }
                }
            }
        }

        // Bottom boundary
        if(ballY >= getHeight()-ballSize) {
            lives--;
            if(lives > 0) {
                ballX = 150;
                ballY = 250;
            } else {
                gameRunning = false;
            }
        }
    }

    public void paint(Graphics g) {
        super.paint(g);
        
        // Draw bricks
        for(int i=0; i<bricks.length; i++) {
            for(int j=0; j<bricks[0].length; j++) {
                if(bricks[i][j]) {
                    g.setColor(Color.ORANGE);
                    g.fillRect(j*55 + 10, i*25 + 50, 50, 20);
                }
            }
        }
        
        // Draw paddle
        g.setColor(Color.BLUE);
        g.fillRect(paddleX, paddleY, paddleWidth, paddleHeight);
        
        // Draw ball
        g.setColor(Color.RED);
        g.fillOval(ballX, ballY, ballSize, ballSize);
        
        // Draw score & lives
        g.setColor(Color.BLACK);
        g.drawString("Score: " + score, 10, 20);
        g.drawString("Lives: " + lives, 300, 20);
        
        if(!gameRunning) {
            g.drawString("GAME OVER!", 150, 180);
        }
    }

    public void keyPressed(KeyEvent e) {
        int key = e.getKeyCode();
        if(key == KeyEvent.VK_LEFT && paddleX > 0) paddleX -= 15;
        if(key == KeyEvent.VK_RIGHT && paddleX < getWidth()-paddleWidth) paddleX += 15;
    }

    public void keyTyped(KeyEvent e) {}
    public void keyReleased(KeyEvent e) {}

    public static void main(String[] args) {
        new SimpleBrickGame();
    }
}
