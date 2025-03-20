# MyProjects
/*
 * Jannatul Mahmud
 * Midterm Project - Cave Diver
 */

import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import javax.swing.JPanel;

class CavePanel extends JPanel {
    private Cave cave;
    private final int CELL_SIZE = 50;

    
    /**
     * Constructor to initialize the cave panel with the given cave.
     *
     * @param cave The cave object to visualize.
     */
    public CavePanel(Cave cave) {
        this.cave = cave;
        setPreferredSize(new Dimension(CELL_SIZE * 10, CELL_SIZE * 10));
    }
  

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        CaveCell[][] grid = cave.getGrid();
        
        for (int i = 0; i < 10; i++) {
            for (int j = 0; j < 10; j++) {
                CaveCell cell = grid[i][j];
                int x = j * CELL_SIZE;
                int y = i * CELL_SIZE;

                // Set color based on escape route or depth
                if (cell.isEscapeRoute()) {
                    g.setColor(Color.RED);
                } else {
                    int blue = 255 - cell.getDepth() * 25;
                    g.setColor(new Color(0, 0, Math.max(blue, 0)));
                }
                g.fillRect(x, y, CELL_SIZE, CELL_SIZE);

                // Draw cell border
                g.setColor(Color.BLACK);
                g.drawRect(x, y, CELL_SIZE, CELL_SIZE);

                // Draw the depth value at the top-left corner of each cell
                g.setColor(Color.WHITE); // Text color (white for better contrast)
                g.drawString(String.valueOf(cell.getDepth()), x + 2, y + 12); // Position the text near the top-left corner
            }
        }
    }

}


/*
 * Jannatul Mahmud
 * Midterm Project - Cave Diver
 */

import javax.swing.*;
import java.awt.*;

public class CaveDiverApp extends JFrame {
    private Cave cave = new Cave();
    private CavePanel cavePanel = new CavePanel(cave);
    private JTextField depthInput = new JTextField(5);

    
    // Constructor to set up the user interface.
    public CaveDiverApp() {
        setupUI();
        setTitle("Cave Diver Escape");
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        pack();
        setLocationRelativeTo(null);
    }

    private void setupUI() {
        setLayout(new BorderLayout(10, 10));
        
        // Header
        add(new JLabel("Help the diver escape! Enter depth rating (1-10):", SwingConstants.CENTER), BorderLayout.NORTH);
        
        // Cave display
        add(cavePanel, BorderLayout.CENTER);

        // Control panel
        JPanel controls = new JPanel();
        controls.add(new JLabel("Enter the Diver's depth rating:"));
        controls.add(depthInput);
        
        JButton escapeBtn = new JButton("Find Escape");
        escapeBtn.addActionListener(e -> findEscape());
        controls.add(escapeBtn);

        JButton newCaveBtn = new JButton("New Cave");
        newCaveBtn.addActionListener(e -> newCave());
        controls.add(newCaveBtn);

        add(controls, BorderLayout.SOUTH);
    }

    
    /**
     * Finds the escape path based on the depth rating entered by the user.
     */
    private void findEscape() {
        try {
            int rating = Integer.parseInt(depthInput.getText());
            boolean found = cave.findEscapePath(rating);
            cavePanel.repaint();
            
            if (!found) 
                JOptionPane.showMessageDialog(this, "No escape path found with depth " + rating + "!", "No Path", JOptionPane.WARNING_MESSAGE);
            
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(this, "Please enter a number between 1-10", "Ouch", JOptionPane.ERROR_MESSAGE);
        }
    }


/*
 * Jannatul Mahmud
 * Midterm Project - Cave Diver
 */

class CaveCell {
    private int row, col, depth, randomNumber;
    private boolean isEscapeRoute;

    /**
     * Constructor to initialize the cave cell with the given row, column, and depth.
     *
     * @param row The row position of the cell in the grid.
     * @param col The column position of the cell in the grid.
     * @param depth The depth level of the cell.
     */
    public CaveCell(int row, int col, int depth) {
        this.row = row;
        this.col = col;
        this.depth = depth;
        this.isEscapeRoute = false;
    }

    /**
     * Gets the depth of the cave cell.
     *
     * @return The depth of the cave cell.
     */
    public int getDepth() {
    	return depth; 
    	}
    
    /**
     * Checks if the cell is part of the escape route.
     *
     * @return true if the cell is part of the escape route, false otherwise.
     */
    public boolean isEscapeRoute() {
    	return isEscapeRoute; 
    	}
    
    /**
     * Sets the escape route status of the cell.
     *
     * @param escapeRoute The escape route status to set.
     */
    public void setEscapeRoute(boolean escapeRoute) {
    	isEscapeRoute = escapeRoute; 
    	}
    // Getter for the random number
    public int getRandomNumber() {
    	return randomNumber; 
    }
}


/*
 * Jannatul Mahmud
 * Midterm Project - Cave Diver
 */

import java.util.Random;

class Cave {
    private static final int SIZE = 10;
    private static final int AIR_LIMIT = 20;
    private CaveCell[][] grid;
    private int airRemaining;

    public Cave() {
        generateNewCave();
        this.airRemaining = AIR_LIMIT;
    }

    public void generateNewCave() {
        grid = new CaveCell[SIZE][SIZE];
        Random rand = new Random();
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                grid[i][j] = new CaveCell(i, j, rand.nextInt(10) + 1);
            }
        }
        this.airRemaining = AIR_LIMIT; // Reset air for a new cave
    }

    
    /**
     * Finds an escape path in the cave, starting from the top-left corner.
     *
     * @param maxDepth The maximum depth allowed to follow during the search.
     * @return true if an escape path is found, false otherwise.
     */
    public boolean findEscapePath(int maxDepth) {
        resetPath();
        return explorePath(0, 0, maxDepth, airRemaining);
    }
    
    

    /**
     * Explores the cave grid recursively to find an escape path.
     *
     * @param row The current row position.
     * @param col The current column position.
     * @param maxDepth The maximum depth allowed during the search.
     * @return true if an escape path is found, false otherwise.
     */
    private boolean explorePath(int row, int col, int maxDepth, int remainingAir) {
        if (row >= SIZE || col >= SIZE || grid[row][col].getDepth() > maxDepth || remainingAir <= 0) 
            return false;

        if (row == SIZE - 1 && col == SIZE - 1) {
            grid[row][col].setEscapeRoute(true);
            return true;
        }

        if (explorePath(row, col + 1, maxDepth, remainingAir - 1) || explorePath(row + 1, col, maxDepth, remainingAir - 1)) {
            grid[row][col].setEscapeRoute(true);
            return true;
        }

        return false;
    }

    private void resetPath() {
        for (CaveCell[] row : grid) 
            for (CaveCell cell : row) 
                cell.setEscapeRoute(false);
    }

    
    /**
     * Gets the cave grid.
     *
     * @return The 2D array representing the cave grid.
     */
    public CaveCell[][] getGrid() {
    	return grid;
    }
}

    
    /**
     * Generates a new cave and repaints the panel.
     */
    private void newCave() {
        cave.generateNewCave();
        cavePanel.repaint();
    }

    
    /**
     * Main method to start the application.
     */
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new CaveDiverApp().setVisible(true));
    }
}
