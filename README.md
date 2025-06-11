import javax.swing.*;
import javax.swing.Timer;
import java.util.Timer;
import java.util.TimerTask;
import java.awt.*;
import java.awt.event.*;
import java.util.*;
import java.util.ArrayList;
import java.io.File;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;

public class SoloLevelingGame extends JFrame implements ActionListener {
    // Game state variables
    private int gems = 100;
    private int playerLevel = 1;
    private int playerHP = 100;
    private int maxHP = 100;
    private int playerAttack = 20;
    private ArrayList<String> inventory;
    private String currentDungeonRank = "";
    private int currentDungeonEnemies = 0;
    private int enemiesKilled = 0;
    private boolean inDungeon = false;
    private boolean inBattle = false;
    private int currentEnemyHP = 0;
    private int maxEnemyHP = 0;
    private String currentEnemyName = "";
    private boolean isBoss = false;
    
    // Intro screen variables
    private boolean showIntro = true;
    private BufferedImage introImage;
    private JPanel introPanel;
    private JButton enterGameBtn;
    
    // GUI Components
    private JPanel mainPanel;
    private JTextArea storyArea;
    private JLabel statusLabel;
    private JLabel gemsLabel;
    private JLabel hpLabel;
    private JButton enterDungeonBtn;
    private JButton inventoryBtn;
    private JButton attackBtn;
    private JButton healBtn;
    private JButton exitDungeonBtn;
    private JProgressBar playerHPBar;
    private JProgressBar enemyHPBar;
    private JLabel enemyLabel;
    private javax.swing.Timer animationTimer;
    private Color backgroundColor = Color.BLACK;
    private boolean isFlashing = false;

    // Background Image variables
    private Map<String, BufferedImage> backgroundImages;
    private String currentBackground = "city"; // Default background

    // Dungeon ranks and their properties
    private String[] dungeonRanks = {"E", "D", "C", "B", "A", "S"};
    private String[] enemyNames = {"Shadow Wolf", "Stone Golem", "Dark Mage", "Flame Dragon", "Ice Giant", "Shadow Monarch"};
    private String[] bossNames = {"Alpha Wolf", "Earth Guardian", "Archmage", "Inferno King", "Frost Titan", "Shadow Emperor"};
    
    public SoloLevelingGame() {
        setTitle("Solo Leveling: Dungeon Hunter");
        setSize(900, 700);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        
        loadIntroImage();
        loadBackgroundImages(); // Load all scene-specific background images
        initializeGame();
        setupIntroScreen();
        
        // Animation timer for visual effects
        animationTimer = new javax.swing.Timer(100, e -> {
            if (isFlashing) {
                backgroundColor = backgroundColor == Color.BLACK ? Color.RED : Color.BLACK;
                if (mainPanel != null) {
                    mainPanel.setBackground(backgroundColor);
                    repaint();
                }
            }
        });
    }
    
    private void loadIntroImage() {
        try {
            // Load your specific uploaded image
            introImage = ImageIO.read(new File("SL/Gemini_Generated_Image_in9wluin9wluin9w.png"));
            System.out.println("Successfully loaded custom intro image!");
        } catch (Exception e) {
            System.out.println("Could not load your custom image: " + e.getMessage());
            // Create a fallback image if your specific image can't be loaded
            introImage = new BufferedImage(800, 600, BufferedImage.TYPE_INT_RGB);
            Graphics2D g2d = introImage.createGraphics();
            
            // Create a similar scene to your uploaded image
            // Dark urban background
            GradientPaint bgGradient = new GradientPaint(0, 0, new Color(20, 30, 50), 
                                                         800, 600, new Color(10, 15, 25));
            g2d.setPaint(bgGradient);
            g2d.fillRect(0, 0, 800, 600);
            
            // Draw buildings silhouettes
            g2d.setColor(new Color(40, 50, 70));
            g2d.fillRect(0, 400, 200, 200);
            g2d.fillRect(600, 350, 200, 250);
            g2d.fillRect(50, 300, 100, 300);
            g2d.fillRect(650, 250, 80, 350);
            
            // Draw the glowing portal
            int portalX = 400, portalY = 300;
            int portalRadius = 120;
            
            // Outer glow
            for (int i = 0; i < 20; i++) {
                float alpha = 0.1f - (i * 0.005f);
                g2d.setColor(new Color(0, 200, 255, (int)(alpha * 255)));
                g2d.fillOval(portalX - portalRadius - i*3, portalY - portalRadius - i*3, 
                           (portalRadius + i*3) * 2, (portalRadius + i*3) * 2);
            }
            
            // Portal rings
            g2d.setColor(new Color(0, 150, 255, 200));
            g2d.setStroke(new BasicStroke(6));
            g2d.drawOval(portalX - portalRadius, portalY - portalRadius, portalRadius * 2, portalRadius * 2);
            
            g2d.setColor(new Color(100, 200, 255, 150));
            g2d.setStroke(new BasicStroke(4));
            g2d.drawOval(portalX - portalRadius + 20, portalY - portalRadius + 20, 
                         (portalRadius - 20) * 2, (portalRadius - 20) * 2);
            
            // Portal center
            RadialGradientPaint portalGradient = new RadialGradientPaint(
                portalX, portalY, portalRadius - 30,
                new float[]{0f, 0.7f, 1f},
                new Color[]{new Color(200, 240, 255, 180), new Color(0, 150, 255, 120), new Color(0, 100, 200, 60)}
            );
            g2d.setPaint(portalGradient);
            g2d.fillOval(portalX - portalRadius + 30, portalY - portalRadius + 30, 
                         (portalRadius - 30) * 2, (portalRadius - 30) * 2);
            
            // Hunter silhouette
            g2d.setColor(new Color(30, 30, 30));
            int hunterX = portalX;
            int hunterY = portalY + 80;
            // Body
            g2d.fillRect(hunterX - 15, hunterY - 40, 30, 60);
            // Head
            g2d.fillOval(hunterX - 12, hunterY - 60, 24, 20);
            // Arms
            g2d.fillRect(hunterX - 25, hunterY - 35, 10, 40);
            g2d.fillRect(hunterX + 15, hunterY - 35, 10, 40);
            // Legs
            g2d.fillRect(hunterX - 12, hunterY + 20, 10, 30);
            g2d.fillRect(hunterX + 2, hunterY + 20, 10, 30);
            
            g2d.dispose();
        }
    }
    
    // --- NEW CODE FOR BACKGROUND IMAGES ---
    private void loadBackgroundImages() {
        backgroundImages = new HashMap<>();
        try {
            // Main city/hub background
            backgroundImages.put("city", ImageIO.read(new File("SL/E-rankemptydungeon.png"))); 
            // Example dungeon backgrounds (you'll replace these with your actual images)
            backgroundImages.put("E_dungeon", ImageIO.read(new File("SL/E-rankemptydungeon.png")));
            backgroundImages.put("D_dungeon", ImageIO.read(new File("SL/Gemini_Generated_Image_badlw7badlw7badl.png")));
            backgroundImages.put("C_dungeon", ImageIO.read(new File("SL/")));
            backgroundImages.put("B_dungeon", ImageIO.read(new File("SL/")));
            backgroundImages.put("A_dungeon", ImageIO.read(new File("SL/")));
            backgroundImages.put("S_dungeon", ImageIO.read(new File("SL/")));
            // You can add more specific backgrounds, e.g., for battle scenes or boss fights
            backgroundImages.put("battle", ImageIO.read(new File("SL/shadow wolf in the dungeon.png"))); // A generic battle background
            System.out.println("Successfully loaded background images.");
        } catch (Exception e) {
            System.err.println("Error loading background images: " + e.getMessage());
            // Fallback for missing images (e.g., use a solid color or a default image)
            // For example, you can create a blank image or use a color as a fallback
            backgroundImages.put("city", createFallbackImage(900, 700, Color.DARK_GRAY));
            backgroundImages.put("E_dungeon", createFallbackImage(900, 700, new Color(50, 0, 0))); // Dark red for E-dungeon
            backgroundImages.put("D_dungeon", createFallbackImage(900, 700, new Color(0, 50, 0))); // Dark green for D-dungeon
            backgroundImages.put("C_dungeon", createFallbackImage(900, 700, new Color(0, 0, 50))); // Dark blue for C-dungeon
            backgroundImages.put("B_dungeon", createFallbackImage(900, 700, new Color(50, 25, 0))); // Dark orange for B-dungeon
            backgroundImages.put("A_dungeon", createFallbackImage(900, 700, new Color(50, 0, 50))); // Dark purple for A-dungeon
            backgroundImages.put("S_dungeon", createFallbackImage(900, 700, new Color(25, 25, 25))); // Very dark gray for S-dungeon
            backgroundImages.put("battle", createFallbackImage(900, 700, new Color(70, 0, 0))); // Darker red for battle
        }
    }

    private BufferedImage createFallbackImage(int width, int height, Color color) {
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        Graphics2D g2d = image.createGraphics();
        g2d.setColor(color);
        g2d.fillRect(0, 0, width, height);
        g2d.dispose();
        return image;
    }

    private void updateBackground(String scene) {
        currentBackground = scene;
        if (mainPanel != null) {
            mainPanel.repaint(); // Request a repaint to draw the new background
        }
    }
    // --- END NEW CODE ---
    
    private void setupIntroScreen() {
        introPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                if (introImage != null) {
                    // Scale and center the image to fill the panel
                    int panelWidth = getWidth();
                    int panelHeight = getHeight();
                    int imgWidth = introImage.getWidth();
                    int imgHeight = introImage.getHeight();
                    
                    // Calculate scaling to fill the panel while maintaining aspect ratio
                    double scaleX = (double)panelWidth / imgWidth;
                    double scaleY = (double)panelHeight / imgHeight;
                    double scale = Math.max(scaleX, scaleY); // Use max to fill completely
                    
                    int scaledWidth = (int)(imgWidth * scale);
                    int scaledHeight = (int)(imgHeight * scale);
                    
                    int x = (panelWidth - scaledWidth) / 2;
                    int y = (panelHeight - scaledHeight) / 2;
                    
                    g.drawImage(introImage, x, y, scaledWidth, scaledHeight, this);
                    
                    // Add a dark overlay for text readability
                    g.setColor(new Color(0, 0, 0, 100));
                    g.fillRect(0, 0, panelWidth, panelHeight);
                }
            }
        };
        
        introPanel.setLayout(new BorderLayout());
        introPanel.setBackground(Color.BLACK);
        
        // Create intro text panel
        JPanel textPanel = new JPanel();
        textPanel.setOpaque(false);
        textPanel.setLayout(new BoxLayout(textPanel, BoxLayout.Y_AXIS));
        
        JLabel titleLabel = new JLabel("SOLO LEVELING: DUNGEON HUNTER");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 28));
        titleLabel.setForeground(new Color(0, 200, 255));
        titleLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        
        JLabel subtitleLabel = new JLabel("The portal to a world full of dungeon creatures awaits...");
        subtitleLabel.setFont(new Font("Arial", Font.ITALIC, 18));
        subtitleLabel.setForeground(Color.WHITE);
        subtitleLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        
        JLabel descLabel = new JLabel("Ancient dungeons have appeared across the world, filled with monsters");
        descLabel.setFont(new Font("Arial", Font.PLAIN, 14));
        descLabel.setForeground(new Color(200, 200, 200));
        descLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        
        JLabel desc2Label = new JLabel("and treasures beyond imagination. Will you step through the portal?");
        desc2Label.setFont(new Font("Arial", Font.PLAIN, 14));
        desc2Label.setForeground(new Color(200, 200, 200));
        desc2Label.setAlignmentX(Component.CENTER_ALIGNMENT);
        
        // Add some spacing
        textPanel.add(Box.createVerticalStrut(30));
        textPanel.add(titleLabel);
        textPanel.add(Box.createVerticalStrut(15));
        textPanel.add(subtitleLabel);
        textPanel.add(Box.createVerticalStrut(20));
        textPanel.add(descLabel);
        textPanel.add(Box.createVerticalStrut(5));
        textPanel.add(desc2Label);
        textPanel.add(Box.createVerticalStrut(40));
        
        // Create enter game button
        enterGameBtn = new JButton("ENTER THE WORLD");
        enterGameBtn.setFont(new Font("Creepster", Font.BOLD, 20));
        enterGameBtn.setBackground(new Color(0, 150, 255));
        enterGameBtn.setForeground(Color.WHITE);
        enterGameBtn.setBorder(BorderFactory.createCompoundBorder
        (
            BorderFactory.createRaisedBevelBorder(),
            BorderFactory.createEmptyBorder(10, 20, 10, 20)
        ));
        enterGameBtn.setFocusPainted(false);
        enterGameBtn.setAlignmentX(Component.CENTER_ALIGNMENT);
        enterGameBtn.setPreferredSize(new Dimension(250, 60));
        enterGameBtn.addActionListener(this);
        
        // Add glowing effect to button
        enterGameBtn.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                enterGameBtn.setBackground(new Color(0, 180, 255));
                enterGameBtn.setBorder(BorderFactory.createCompoundBorder(
                    BorderFactory.createLineBorder(new Color(100, 200, 255), 2),
                    BorderFactory.createEmptyBorder(8, 18, 8, 18)
                ));
            }
            
            @Override
            public void mouseExited(MouseEvent e) {
                enterGameBtn.setBackground(new Color(0, 150, 255));
                enterGameBtn.setBorder(BorderFactory.createCompoundBorder(
                    BorderFactory.createRaisedBevelBorder(),
                    BorderFactory.createEmptyBorder(10, 20, 10, 20)
                ));
            }
        });
        
        textPanel.add(enterGameBtn);
        textPanel.add(Box.createVerticalStrut(50));
        
        introPanel.add(textPanel, BorderLayout.SOUTH);
        
        add(introPanel);
    }
    
    private void startMainGame() {
        showIntro = false;
        remove(introPanel);
        setupGUI();
        updateDisplay();
        updateBackground("city"); // Set initial background to city/hub
        revalidate();
        repaint();
    }
    
    private void initializeGame() {
        inventory = new ArrayList<>();
        inventory.add("Basic Sword");
        inventory.add("Health Potion");
    }
    
    private void setupGUI() {
        mainPanel = new JPanel() { 
            
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                BufferedImage bg = backgroundImages.get(currentBackground);
                if (bg != null) {
                    // Scale and center the image to fill the panel
                    int panelWidth = getWidth();
                    int panelHeight = getHeight();
                    int imgWidth = bg.getWidth();
                    int imgHeight = bg.getHeight();
                    
                    double scaleX = (double)panelWidth / imgWidth;
                    double scaleY = (double)panelHeight / imgHeight;
                    double scale = Math.max(scaleX, scaleY); 
                    
                    int scaledWidth = (int)(imgWidth * scale);
                    int scaledHeight = (int)(imgHeight * scale);
                    
                    int x = (panelWidth - scaledWidth) / 2;
                    int y = (panelHeight - scaledHeight) / 2;
                    
                    g.drawImage(bg, x, y, scaledWidth, scaledHeight, this);
                }
            }
        };
        mainPanel.setLayout(new BorderLayout());
        // mainPanel.setBackground(Color.BLACK); // Background is now handled by paintComponent
        
        // Top status panel
        JPanel statusPanel = new JPanel(new GridLayout(3, 1));
        statusPanel.setBackground(new Color(0, 0, 0, 150)); // Semi-transparent black for readability
        statusPanel.setBorder(BorderFactory.createTitledBorder(
            BorderFactory.createLineBorder(Color.CYAN, 2), "Hunter Status", 
            0, 0, new Font("Arial", Font.BOLD, 14), Color.CYAN));
        
        statusLabel = new JLabel("Level: " + playerLevel + " | Rank: E-Class Hunter", JLabel.CENTER);
        statusLabel.setForeground(Color.WHITE);
        statusLabel.setFont(new Font("Arial", Font.BOLD, 16));
        
        gemsLabel = new JLabel("Gems: " + gems, JLabel.CENTER);
        gemsLabel.setForeground(Color.YELLOW);
        gemsLabel.setFont(new Font("Arial", Font.BOLD, 14));
        
        hpLabel = new JLabel("HP: " + playerHP + "/" + maxHP, JLabel.CENTER);
        hpLabel.setForeground(Color.GREEN);
        hpLabel.setFont(new Font("Arial", Font.BOLD, 14));
        
        statusPanel.add(statusLabel);
        statusPanel.add(gemsLabel);
        statusPanel.add(hpLabel);
        
        // Center story area
        storyArea = new JTextArea();
        storyArea.setEditable(false);
        storyArea.setBackground(new Color(0, 0, 0, 180)); // Semi-transparent black
        storyArea.setForeground(Color.WHITE);
        storyArea.setFont(new Font("Courier New", Font.PLAIN, 14));
        storyArea.setBorder(BorderFactory.createLineBorder(Color.BLUE, 2));
        JScrollPane scrollPane = new JScrollPane(storyArea);
        scrollPane.setPreferredSize(new Dimension(600, 300));
        scrollPane.setOpaque(false); // Make scroll pane transparent
        scrollPane.getViewport().setOpaque(false); // Make viewport transparent
        
        // HP Bars panel
        JPanel barsPanel = new JPanel(new GridLayout(2, 1, 5, 5));
        barsPanel.setOpaque(false); // Make bars panel transparent
        
        playerHPBar = new JProgressBar(0, maxHP);
        playerHPBar.setValue(playerHP);
        playerHPBar.setStringPainted(true);
        playerHPBar.setString("Your HP");
        playerHPBar.setForeground(Color.GREEN);
        playerHPBar.setBackground(Color.DARK_GRAY);
        
        enemyHPBar = new JProgressBar(0, 100);
        enemyHPBar.setValue(0);
        enemyHPBar.setStringPainted(true);
        enemyHPBar.setString("Enemy HP");
        enemyHPBar.setForeground(Color.RED);
        enemyHPBar.setBackground(Color.DARK_GRAY);
        enemyHPBar.setVisible(false);
        
        barsPanel.add(playerHPBar);
        barsPanel.add(enemyHPBar);
        
        // Enemy label
        enemyLabel = new JLabel("", JLabel.CENTER);
        enemyLabel.setForeground(Color.RED);
        enemyLabel.setFont(new Font("Arial", Font.BOLD, 16));
        enemyLabel.setVisible(false);
        
        // Bottom button panel
        JPanel buttonPanel = new JPanel(new FlowLayout());
        buttonPanel.setBackground(new Color(0, 0, 0, 150)); // Semi-transparent black
        
        enterDungeonBtn = new JButton("Enter Dungeon");
        inventoryBtn = new JButton("Inventory");
        attackBtn = new JButton("Attack");
        healBtn = new JButton("Heal (Cost: 10 gems)");
        exitDungeonBtn = new JButton("Exit Dungeon");
        
        styleButton(enterDungeonBtn);
        styleButton(inventoryBtn);
        styleButton(attackBtn);
        styleButton(healBtn);
        styleButton(exitDungeonBtn);
        
        enterDungeonBtn.addActionListener(this);
        inventoryBtn.addActionListener(this);
        attackBtn.addActionListener(this);
        healBtn.addActionListener(this);
        exitDungeonBtn.addActionListener(this);
        
        buttonPanel.add(enterDungeonBtn);
        buttonPanel.add(inventoryBtn);
        buttonPanel.add(attackBtn);
        buttonPanel.add(healBtn);
        buttonPanel.add(exitDungeonBtn);
        
        // Initially hide battle buttons
        attackBtn.setVisible(false);
        exitDungeonBtn.setVisible(false);
        
        // Assemble main panel
        JPanel centerPanel = new JPanel(new BorderLayout());
        centerPanel.setOpaque(false); // Make center panel transparent
        centerPanel.add(scrollPane, BorderLayout.CENTER);
        centerPanel.add(barsPanel, BorderLayout.SOUTH);
        
        JPanel topPanel = new JPanel(new BorderLayout());
        topPanel.setOpaque(false); // Make top panel transparent
        topPanel.add(statusPanel, BorderLayout.CENTER);
        topPanel.add(enemyLabel, BorderLayout.SOUTH);
        
        mainPanel.add(topPanel, BorderLayout.NORTH);
        mainPanel.add(centerPanel, BorderLayout.CENTER);
        mainPanel.add(buttonPanel, BorderLayout.SOUTH);
        
        add(mainPanel);
        
        displayWelcomeMessage();
    }
    
    private void styleButton(JButton button) {
        button.setBackground(Color.BLUE);
        button.setForeground(Color.WHITE);
        button.setFont(new Font("Arial", Font.BOLD, 12));
        button.setBorder(BorderFactory.createRaisedBevelBorder());
        button.setFocusPainted(false);
    }
    
    private void displayWelcomeMessage() {
        storyArea.setText("=== WELCOME TO THE WORLD OF DUNGEON HUNTERS ===\n\n" +
            "You stand before a mysterious glowing portal that has appeared in the city.\n" +
            "Through it, you can sense a world filled with dangerous dungeon creatures,\n" +
            "ancient magic, and untold treasures waiting to be discovered.\n\n" +
            "As a newly awakened Hunter, you possess the power to enter these dungeons\n" +
            "and grow stronger with each battle. But beware - the creatures within are\n" +
            "not to be underestimated.\n\n" +
            "DUNGEON RANKS:\n" +
            "• E-Rank: Shadow Wolves and basic monsters\n" +
            "• D-Rank: Stone Golems and stronger foes\n" +
            "• C-Rank: Dark Mages with magical abilities\n" +
            "• B-Rank: Flame Dragons breathing fire\n" +
            "• A-Rank: Ice Giants of immense power\n" +
            "• S-Rank: Shadow Monarchs - the ultimate challenge\n\n" +
            "Each dungeon contains multiple monsters and ends with a powerful boss.\n" +
            "Defeating creatures rewards you with gems to upgrade your equipment.\n\n" +
            "Are you ready to step through the portal and face the dungeon creatures?\n" +
            "Click 'Enter Dungeon' when you're prepared for battle!\n\n" +
            "Use 'Inventory' to check your items and purchase new equipment.\n" +
            "Good luck, Hunter! The world of dungeons awaits your courage!");
    }
    
    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == enterGameBtn) {
            startMainGame();
        } else if (e.getSource() == enterDungeonBtn) {
            enterDungeon();
        } else if (e.getSource() == inventoryBtn) {
            showInventory();
        } else if (e.getSource() == attackBtn) {
            attackEnemy();
        } else if (e.getSource() == healBtn) {
            healPlayer();
        } else if (e.getSource() == exitDungeonBtn) {
            exitDungeon();
        }
    }
    
    private void enterDungeon() {
        if (inDungeon) {
            JOptionPane.showMessageDialog(this, "You are already in a dungeon!");
            return;
        }
        
        // Generate random dungeon
        Random rand = new Random();
        currentDungeonRank = dungeonRanks[rand.nextInt(dungeonRanks.length)];
        
        // Set dungeon properties based on rank
        switch (currentDungeonRank) {
            case "E": currentDungeonEnemies = 3 + rand.nextInt(3); break;
            case "D": currentDungeonEnemies = 5 + rand.nextInt(3); break;
            case "C": currentDungeonEnemies = 7 + rand.nextInt(4); break;
            case "B": currentDungeonEnemies = 10 + rand.nextInt(5); break;
            case "A": currentDungeonEnemies = 15 + rand.nextInt(5); break;
            case "S": currentDungeonEnemies = 20 + rand.nextInt(10); break;
        }
        
        inDungeon = true;
        enemiesKilled = 0;
        
        generateDungeonStory();
        spawnEnemy();
        
        enterDungeonBtn.setVisible(false);
        exitDungeonBtn.setVisible(true);
        attackBtn.setVisible(true);
        enemyHPBar.setVisible(true);
        enemyLabel.setVisible(true);

        updateBackground(currentDungeonRank + "_dungeon"); // Update background based on dungeon rank
        updateDisplay();
    }
    
    private void generateDungeonStory() {
        String[] stories = {
            "You step through the glowing portal into a dark cavern filled with the howls of Shadow Wolves. Ancient runes pulse with dark energy on the stone walls.",
            "The portal opens to reveal a massive underground fortress where Stone Golems patrol the halls. Their heavy footsteps echo through the chambers.",
            "You enter a mystical realm where Dark Mages have corrupted the very air with their magic. Spells crackle and spark in the twisted atmosphere.",
            "Before you lies a volcanic dungeon where Flame Dragons soar overhead, breathing torrents of fire. The heat is almost unbearable.",
            "An icy wasteland spreads before you, home to massive Ice Giants whose frozen breath can kill in seconds. The cold cuts through your very soul.",
            "You have entered the Shadow Realm - domain of the Shadow Monarchs. Here, darkness itself is alive and hungry for mortal souls."
        };
        
        Random rand = new Random();
        String story = stories[getRankIndex(currentDungeonRank)];
        
        storyArea.setText("=== DUNGEON PORTAL ACTIVATED ===\n" +
            "Rank: " + currentDungeonRank + "-Rank Dungeon\n" +
            "Dungeon creatures to defeat: " + currentDungeonEnemies + "\n\n" +
            story + "\n\n" +
            "Your mission: Eliminate all dungeon creatures including the boss to claim your rewards!\n" +
            "Remember - each creature defeated makes you stronger and brings valuable gems.\n\n");
    }
    
    private void spawnEnemy() {
        if (enemiesKilled >= currentDungeonEnemies) {
            completeDungeon();
            return;
        }
        
        Random rand = new Random();
        
        // Check if this should be the boss (last enemy)
        isBoss = (enemiesKilled == currentDungeonEnemies - 1);
        
        if (isBoss) {
            currentEnemyName = bossNames[getRankIndex(currentDungeonRank)] + " (BOSS)";
            maxEnemyHP = 50 + (getRankIndex(currentDungeonRank) * 30);
        } else {
            currentEnemyName = enemyNames[getRankIndex(currentDungeonRank)];
            maxEnemyHP = 20 + (getRankIndex(currentDungeonRank) * 10) + rand.nextInt(20);
        }
        
        currentEnemyHP = maxEnemyHP;
        inBattle = true;
        
        enemyHPBar.setMaximum(maxEnemyHP);
        enemyHPBar.setValue(currentEnemyHP);
        enemyHPBar.setString(currentEnemyName + " HP");
        enemyLabel.setText("⚔️ " + currentEnemyName + " emerges from the shadows!");
        
        String battleText = "\n=== CREATURE ENCOUNTER ===\n" +
            "A fearsome " + currentEnemyName + " blocks your path!\n" +
            "Creature HP: " + currentEnemyHP + "/" + maxEnemyHP + "\n" +
            (isBoss ? "🔥 This is the dungeon boss - prepare for an epic battle! 🔥\n" : "") +
            "Ready your weapon and prepare for combat!\n\n";
        
        storyArea.append(battleText);
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
        
        // Flash effect for enemy appearance
        flashScreen();
    }
    
    private void attackEnemy() {
        if (!inBattle) {
            spawnEnemy();
            return;
        }
        
        Random rand = new Random();
        int damage = playerAttack + rand.nextInt(15);
        currentEnemyHP -= damage;
        
        String attackText = "💥 You strike " + currentEnemyName + " for " + damage + " damage!\n";
        storyArea.append(attackText);
        
        if (currentEnemyHP <= 0) {
            currentEnemyHP = 0;
            defeatedEnemy();
        } else {
            // Enemy counter-attack
            int enemyDamage = 5 + getRankIndex(currentDungeonRank) * 3 + rand.nextInt(10);
            if (isBoss) enemyDamage *= 2;
            
            playerHP -= enemyDamage;
            if (playerHP < 0) playerHP = 0;
            
            String counterText = "💀 " + currentEnemyName + " strikes back for " + enemyDamage + " damage!\n\n";
            storyArea.append(counterText);
            
            if (playerHP <= 0) {
                gameOver();
                return;
            }
        }
        
        enemyHPBar.setValue(currentEnemyHP);
        updateDisplay();
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
        
        // Flash effect for combat
        flashScreen();
    }
    
    private void defeatedEnemy() {
        enemiesKilled++;
        inBattle = false;
        
        Random rand = new Random();
        int gemsEarned = (5 + getRankIndex(currentDungeonRank) * 5) + rand.nextInt(10);
        if (isBoss) gemsEarned *= 3;
        
        gems += gemsEarned;
        
        String victoryText = "🏆 " + currentEnemyName + " has been defeated!\n" +
            "💎 You earned " + gemsEarned + " gems from the fallen creature!\n" +
            "Creatures defeated: " + enemiesKilled + "/" + currentDungeonEnemies + "\n\n";
        
        storyArea.append(victoryText);
        
        if (isBoss) {
            storyArea.append("🎊 DUNGEON BOSS DEFEATED! The portal begins to destabilize...\n\n");
        } else {
            storyArea.append("You sense another creature lurking deeper in the dungeon...\n\n");
        }
        
        enemyLabel.setText("Searching for the next creature...");
        
        // Auto-spawn next enemy after a brief delay
        javax.swing.Timer spawnTimer = new javax.swing.Timer(2000, e -> {
            spawnEnemy();
            ((javax.swing.Timer)e.getSource()).stop();
        });
        spawnTimer.setRepeats(false);
        spawnTimer.start();
        
        updateDisplay();
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
    }
    
    private void completeDungeon() {
        inDungeon = false;
        inBattle = false;
        
        // Bonus gems for completing dungeon
        int bonusGems = getRankIndex(currentDungeonRank) * 20 + 50;
        gems += bonusGems;
        
        // Level up chance
        Random rand = new Random();
        if (rand.nextInt(100) < 30) { // 30% chance
            playerLevel++;
            maxHP += 20;
            playerHP = maxHP; // Full heal on level up
            playerAttack += 5;
            
            storyArea.append("🌟 LEVEL UP! You are now level " + playerLevel + "!\n" +
                "HP increased to " + maxHP + "!\n" +
                "Attack increased to " + playerAttack + "!\n\n");
        }
        
        String completionText = "🎉 DUNGEON CLEARED! 🎉\n" +
            "Rank " + currentDungeonRank + " dungeon successfully completed!\n" +
            "💎 Bonus gems earned: " + bonusGems + "\n" +
            "💰 Total gems: " + gems + "\n\n" +
            "The dungeon portal closes behind you. You return to the surface, stronger than before.\n" +
            "Ready for your next adventure?\n\n";
        
        storyArea.append(completionText);
        
        enterDungeonBtn.setVisible(true);
        exitDungeonBtn.setVisible(false);
        attackBtn.setVisible(false);
        enemyHPBar.setVisible(false);
        enemyLabel.setVisible(false);
        
        updateBackground("city"); // Return to city background after dungeon
        updateDisplay();
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
    }
    
    private void gameOver() {
        storyArea.append("\n💀 GAME OVER 💀\n" +
            "You have been defeated in the dungeon!\n" +
            "Don't give up - every Hunter faces defeat before becoming strong!\n\n");
        
        // Reset player
        playerHP = maxHP;
        inDungeon = false;
        inBattle = false;
        
        enterDungeonBtn.setVisible(true);
        exitDungeonBtn.setVisible(false);
        attackBtn.setVisible(false);
        enemyHPBar.setVisible(false);
        enemyLabel.setVisible(false);
        
        updateBackground("city"); // Return to city background on game over
        updateDisplay();
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
    }
    
    private void healPlayer() {
        if (gems < 10) {
            JOptionPane.showMessageDialog(this, "Not enough gems! You need 10 gems to heal.");
            return;
        }
        
        if (playerHP >= maxHP) {
            JOptionPane.showMessageDialog(this, "Your HP is already full!");
            return;
        }
        
        gems -= 10;
        int healAmount = 30 + new Random().nextInt(20);
        playerHP += healAmount;
        if (playerHP > maxHP) playerHP = maxHP;
        
        storyArea.append("💚 You used a healing item and recovered " + healAmount + " HP!\n\n");
        updateDisplay();
        storyArea.setCaretPosition(storyArea.getDocument().getLength());
    }
    
    private void exitDungeon() {
        if (!inDungeon) return;
        
        int confirm = JOptionPane.showConfirmDialog(this, 
            "Are you sure you want to exit the dungeon? You'll lose progress but keep gems earned so far.",
            "Exit Dungeon", JOptionPane.YES_NO_OPTION);
        
        if (confirm == JOptionPane.YES_OPTION) {
            inDungeon = false;
            inBattle = false;
            
            storyArea.append("\n🚪 You escaped from the dungeon!\n" +
                "Progress lost, but you live to fight another day.\n\n");
            
            enterDungeonBtn.setVisible(true);
            exitDungeonBtn.setVisible(false);
            attackBtn.setVisible(false);
            enemyHPBar.setVisible(false);
            enemyLabel.setVisible(false);
            
            updateBackground("city"); // Return to city background when exiting dungeon
            updateDisplay();
            storyArea.setCaretPosition(storyArea.getDocument().getLength());
        }
    }
    
    private void showInventory() {
        StringBuilder inv = new StringBuilder("=== HUNTER INVENTORY ===\n\n");
        inv.append("💎 Gems: ").append(gems).append("\n\n");
        inv.append("🎒 Items:\n");
        
        for (String item : inventory) {
            inv.append("• ").append(item).append("\n");
        }
        
        inv.append("\n🛒 SHOP:\n");
        inv.append("• Iron Sword (+10 attack) - 100 gems\n");
        inv.append("• Steel Armor (+20 max HP) - 150 gems\n");
        inv.append("• Magic Ring (+15 attack) - 200 gems\n");
        inv.append("• Dragon Blade (+25 attack) - 500 gems\n");
        
        String[] options = {"Buy Iron Sword (100g)", "Buy Steel Armor (150g)", 
                            "Buy Magic Ring (200g)", "Buy Dragon Blade (500g)", "Close"};
        
        int choice = JOptionPane.showOptionDialog(this, inv.toString(), "Inventory & Shop",
            JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE, null, options, options[4]);
        
        switch (choice) {
            case 0: buyItem("Iron Sword", 100, 10, "attack"); break;
            case 1: buyItem("Steel Armor", 150, 20, "health"); break;
            case 2: buyItem("Magic Ring", 200, 15, "attack"); break;
            case 3: buyItem("Dragon Blade", 500, 25, "attack"); break;
        }
    }
    
    private void buyItem(String itemName, int cost, int bonus, String type) {
        if (gems < cost) {
            JOptionPane.showMessageDialog(this, "Not enough gems! You need " + cost + " gems.");
            return;
        }
        
        if (inventory.contains(itemName)) {
            JOptionPane.showMessageDialog(this, "You already own this item!");
            return;
        }
        
        gems -= cost;
        inventory.add(itemName);
        
        if (type.equals("attack")) {
            playerAttack += bonus;
        } else if (type.equals("health")) {
            maxHP += bonus;
            playerHP += bonus;
        }
        
        JOptionPane.showMessageDialog(this, "Purchased " + itemName + "!\n" +
            "Your " + (type.equals("attack") ? "attack" : "max HP") + " increased by " + bonus + "!");
        updateDisplay();
    }
    
    private int getRankIndex(String rank) {
        for (int i = 0; i < dungeonRanks.length; i++) {
            if (dungeonRanks[i].equals(rank)) {
                return i;
            }
        }
        return 0; // Default to E-rank if something goes wrong
    }
    
    private void updateDisplay() {
        statusLabel.setText("Level: " + playerLevel + " | Rank: E-Class Hunter"); // You might want to update Hunter Rank based on level
        gemsLabel.setText("Gems: " + gems);
        hpLabel.setText("HP: " + playerHP + "/" + maxHP);
        playerHPBar.setMaximum(maxHP);
        playerHPBar.setValue(playerHP);
        playerHPBar.setString("Your HP: " + playerHP + "/" + maxHP);
        
        if (inBattle) {
            enemyHPBar.setString(currentEnemyName + " HP: " + currentEnemyHP + "/" + maxEnemyHP);
        } else {
            enemyHPBar.setString("Enemy HP"); // Reset string when not in battle
        }
    }
    
    private void flashScreen() {
        isFlashing = true;
        animationTimer.start();
        javax.swing.Timer stopFlashTimer = new javax.swing.Timer(300, e -> {
            isFlashing = false;
            animationTimer.stop();
            if (mainPanel != null) {
                // Ensure background returns to normal (or the current scene's background)
                mainPanel.setBackground(Color.BLACK); 
                repaint();
            }
            ((javax.swing.Timer)e.getSource()).stop();
        });
        stopFlashTimer.setRepeats(false);
        stopFlashTimer.start();
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new SoloLevelingGame().setVisible(true);
        });
    }
}
