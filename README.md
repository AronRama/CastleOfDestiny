import java.util.Scanner;

public class Game {
    private Room currentRoom;
    private Player player;

    public Game() {
        player = new Player("Hero", 10);
        createRooms();
    }

    private void createRooms() {
        Room entranceHall = new Room("in the Entrance Hall of the castle");
        Room library = new Room("in a grand Library filled with ancient books");
        Room throneRoom = new Room("in the majestic Throne Room", true);
        Room dungeon = new Room("in a dark, damp Dungeon");
        Room armory = new Room("in the Armory, full of ancient weapons");

        // Raumübergänge festlegen
        entranceHall.setExits(null, library, dungeon, armory);
        library.setExits(null, null, throneRoom, entranceHall);
        throneRoom.setExits(null, null, null, library);
        dungeon.setExits(entranceHall, null, null, null);
        armory.setExits(null, entranceHall, null, null);

        // Gegenstände und Charaktere hinzufügen
        library.addItem(new Item("Magic Book", 2));
        armory.addItem(new Item("Sword", 3));
        dungeon.addCharacter(new Character("Mysterious Stranger", "Sei Vorsichtig in dem naechsten Raum vielleicht ist da ein Monster..."));

        currentRoom = entranceHall;
    }

    public void play() {
        System.out.println("Willkommen zur CastleOfDestiny!");
        System.out.println("Type 'help' if you need help.");
        System.out.println();
        System.out.println(currentRoom.getLongDescription());

        Scanner scanner = new Scanner(System.in);
        boolean finished = false;

        while (!finished) {
            System.out.print("> "); // Eingabeaufforderung
            String command = scanner.nextLine();
            finished = processCommand(command);
        }
        System.out.println("Danke fuers Spielen. Tschau!");
        scanner.close();
    }

    private boolean processCommand(String command) {
        if (command.equals("help")) {
            printHelp();
        } else if (command.startsWith("go")) {
            goRoom(command.substring(3));
        } else if (command.startsWith("take")) {
            player.takeItem(currentRoom.getItem(command.substring(5)));
        } else if (command.startsWith("talk")) {
            currentRoom.talkToCharacter();
        } else if (command.equals("inventory")) {
            player.showInventory();
        } else if (command.equals("quit")) {
            return true;
        } else {
            System.out.println("I don't understand that command.");
        }
        return false;
    }

    private void printHelp() {
        System.out.println("Your command words are: go [direction], take [item], talk, inventory, quit");
    }

    private void goRoom(String direction) {
        Room nextRoom = currentRoom.getExit(direction);
        if (nextRoom == null) {
            System.out.println("There is no door!");
        } else {
            currentRoom = nextRoom;
            System.out.println(currentRoom.getLongDescription());
            if (currentRoom.isWinningRoom()) {
                System.out.println("Congratulations! You've reached the Throne Room and won the game!");
                System.exit(0);
            }
        }
    }
}


public class Character {
    private String name;
    private String dialogue;

    public Character(String name, String dialogue) {
        this.name = name;
        this.dialogue = dialogue;
    }

    public String getName() {
        return name;
    }

    public String getDialogue() {
        return dialogue;
    }
}


public class CastleOfDestinyGame {
    public static void main(String[] args) {
        Game game = new Game();
        game.play();
    }
}


import java.util.HashMap;

public class Room {
    private String description;
    private HashMap<String, Room> exits;
    private Item item;
    private Character character;
    private boolean isWinningRoom;

    public Room(String description) {
        this(description, false);
    }

    public Room(String description, boolean isWinningRoom) {
        this.description = description;
        this.isWinningRoom = isWinningRoom;
        exits = new HashMap<>();
    }

    public void setExits(Room north, Room east, Room south, Room west) {
        if (north != null) exits.put("north", north);
        if (east != null) exits.put("east", east);
        if (south != null) exits.put("south", south);
        if (west != null) exits.put("west", west);
    }

    public Room getExit(String direction) {
        return exits.get(direction);
    }

    public void addItem(Item item) {
        this.item = item;
    }

    public Item getItem(String itemName) {
        if (item != null && item.getName().equalsIgnoreCase(itemName)) {
            Item temp = item;
            item = null; // Gegenstand wird aus dem Raum entfernt
            return temp;
        }
        System.out.println("Item not found in this room.");
        return null;
    }

    public void addCharacter(Character character) {
        this.character = character;
    }

    public void talkToCharacter() {
        if (character != null) {
            System.out.println(character.getDialogue());
        } else {
            System.out.println("No one to talk to here.");
        }
    }

    public String getLongDescription() {
        return "You are " + description + ".\n" + getExitString() + getItemDescription() + getCharacterDescription();
    }

    private String getExitString() {
        StringBuilder exitString = new StringBuilder("Exits:");
        for (String exit : exits.keySet()) {
            exitString.append(" ").append(exit);
        }
        return exitString.toString();
    }

    private String getItemDescription() {
        return item != null ? "\nYou see a " + item.getName() + " here." : "";
    }

    private String getCharacterDescription() {
        return character != null ? "\nYou see " + character.getName() + " here." : "";
    }

    public boolean isWinningRoom() {
        return isWinningRoom;
    }
}


import java.util.ArrayList;

public class Player {
    private String name;
    private int maxWeight;
    private ArrayList<Item> inventory;
    private int currentWeight;

    public Player(String name, int maxWeight) {
        this.name = name;
        this.maxWeight = maxWeight;
        this.inventory = new ArrayList<>();
        this.currentWeight = 0;
    }

    public void takeItem(Item item) {
        if (item == null) return;

        if (currentWeight + item.getWeight() <= maxWeight) {
            inventory.add(item);
            currentWeight += item.getWeight();
            System.out.println("You picked up: " + item.getName());
        } else {
            System.out.println("Item is too heavy to carry.");
        }
    }

    public void showInventory() {
        System.out.println("Inventory:");
        for (Item item : inventory) {
            System.out.println("- " + item.getName() + " (Weight: " + item.getWeight() + ")");
        }
    }
}


public class Item {
    private String name;
    private int weight;

    public Item(String name, int weight) {
        this.name = name;
        this.weight = weight;
    }

    public String getName() {
        return name;
    }

    public int getWeight() {
        return weight;
    }
}
