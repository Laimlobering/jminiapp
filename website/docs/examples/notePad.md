# NotePad Example Application
Build a simple note-taking application to learn about text management, state persistence, and import/export operations.

**Features:**
- **Text Management**
- **Real-time Preview**
- **State Persistence**
- **JSON Import/Export**
- **Interactive menu system**

**Source Code:** [examples/notePad](https://github.com/jminiapp/jminiapp/tree/main/examples/notePad)


### Key Concepts Demonstrated

- Application lifecycle (initialize, run, shutdown)
- In-memory state management
- JSON format adapter implementation
- Framework-based import/export

### Quick Start

```bash
cd examples/notePad
mvn clean install
mvn exec:java
```

## Code Highlights

### State Model:

Our `NotePadStates` model needs to store the note text and provide basic operations.

```java
public class NotePadStates {
    private String note;

    public NotePadStates() {
        this.note = "";
    }

    public NotePadStates(String note) {
        this.note = note;
    }

    public String getNote() {
        return note;
    }

    public void setNote(String note) {
        this.note = note;
    }

    public String appendNote(String additionalText) {
        if (additionalText == null) return this.note;
        return this.note += additionalText;
    }

    public String clearNote() {
        return this.note = "";
    }

    public int getNoteLength() {
        return this.note.length();
    }

    @Override
    public String toString() {
        return "NotePadStates{note='" + note + "'}";
    }

}
```

### Adapter
We use the JSONAdapter pattern. JMiniApp handles the serialization of the note field automatically.
```java
/**
 * JSON adapter for CounterState objects.
 *
 * <p>This adapter enables the NotePad app to import and export notepad state
 * to/from JSON files. It leverages the framework's JSONAdapter interface which
 * provides default implementations for serialization using Gson.</p>
 *
 * <p>Example JSON format:</p>
 * <pre>
 * [
 *   {
 *     "note": Pongame 10 Profe
 *   }
 * ]
 * </pre>
 */
public class NotePadJSONAdapter implements JSONAdapter<NotePadStates>{
    @Override
    public Class<NotePadStates> getstateClass() {
        return NotePadStates.class;
    }
}
```

### Application
The core logic resides in `handleUserInput()`. We manage a single note with various operations and provide import/export functionality.

```java
/**
 * A simple application that represents a notepad
 * 
 * The app allows user to:
 * - Create a new note
 * - Append text to the note
 * - Clear the note
 * - View the current note text
 * - Get the length of the note text
 */
public class NotePadApp extends JMiniApp {
    private Scanner scanner;
    private NotePadStates notePad;
    private boolean running;

    public NotePadApp(JMiniAppConfig config) {
        super(config);
    }

    @Override
    protected void initialize() {
        System.out.println("\n=== NotePad App ===");
        System.out.println("Welcome to the NotePad App");
        scanner = new Scanner(System.in);
        running = true;

        List<NotePadStates> data = context.getData();
        if (data != null && !data.isEmpty()) {
            notePad = data.get(0);
            System.out.println("Loaded existing note: " + notePad.getNote());
        } else {
            notePad = new NotePadStates();
            System.out.println("Starting with a new empty note.");
        }
    }

    @Override
    protected void run() {
        while (running) {
            displayMenu();
            handleUserInput();
        }
    }

    @Override
    protected void shutdown() {
        List<NotePadStates> data = List.of(notePad);
        context.setData(data);

        scanner.close();
        System.out.println("\nFinal note text: " + notePad.getNote());
        System.out.println("Goodbye!");
    }

    private void displayMenu(){
        System.out.println("\n--- Current Note: " + notePad.getNote() + " ---");
        System.out.println("1. Append Text");
        System.out.println("2. Clear Note");
        System.out.println("3. Get Note Length");
        System.out.println("4. Export to JSON file");
        System.out.println("5. Import from JSON file");
        System.out.println("6. Exit");
        System.out.println("\nChoose an option: ");

    }

    private void handleUserInput() {
        try {
            String input = scanner.nextLine().trim();

            switch (input) {
                case "1":
                    System.out.println("Enter text to append:");
                    String text = scanner.nextLine();
                    notePad.appendNote(text);
                    System.out.println("Append Note: " + text);
                    break;

                case "2":
                    notePad.clearNote();
                    System.out.println("Note cleared");
                    break;

                case "3":
                    int noteLength= notePad.getNoteLength();
                    System.out.println("Note length: " + noteLength);
                    break;

                case "4":
                    exportToFile();
                    break;

                case "5":
                    importFromFile();
                    break;

                case "6":
                    running = false;
                    System.out.println("\nExiting...");
                    break;

                default:
                    System.out.println("Invalid option. Please choose 1-6.");
            }
        } catch (Exception e) {
            System.out.println("Exception caught during command handling:");
            e.printStackTrace();
        }
    }

    private void exportToFile() {
        try {
            context.setData(List.of(notePad));

            context.exportData("json");
            System.out.println("NotePad state exported successfully to: NotePad.json");
        } catch (IOException e) {
            System.out.println("Error exporting file: " + e.getMessage());
        }
    }

    private void importFromFile() {
        try {

            context.importData("json");

            List<NotePadStates> data = context.getData();
            if (data != null && !data.isEmpty()) {
                notePad = data.get(0);
                System.out.println("NotePad state imported successfully!");
                System.out.println("New text: " + notePad.getNote());
            } else {
                System.out.println("Error: No data found in file.");
            }
        } catch (IOException e) {
             System.out.println("Import failed with exception:");
            e.printStackTrace();
        }
    }

}
```

### Runner
Finally, bootstrap the application.
```java
public class NotePadAppRunner {
    public static void main(String[] args) {
        JMiniAppRunner
            .forApp(NotePadApp.class)
            .withState(NotePadStates.class)
            .withAdapters(new NotePadJSONAdapter())
            //.withResourcesPath("test-data/")  // Custom import/export path
            .named("NotePad")
            .run(args);
    }
}

```
This example demonstrates how to build a stateful text management application where the state (note content) changes based on user interaction and persists automatically between sessions with import/export capabilities.