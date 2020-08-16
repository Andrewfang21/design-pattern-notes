# Design Pattern Notes

### Facade Pattern

- <b>Facades does not encapsulate the subsystem classes</b>; they merely <b>provide a simplified interface to their functionality</b>.
- Example:
  Suppose you want to watch a movie at your own home theater. There are some required steps to be done beforehand, such as:
  - Prepre your popper
  - Turning on projector
  - Turning on DVD
  - and so on...
- Sample Code (Taken from Head-First Design Pattern pg. 265):

  ```java
    public class HomeTheaterFacade {
      Amplifier amp;
      Tuner tuner;
      DvdPlayer dvd;
      CdPlayer cd;
      Projector projector;
      TheaterLights lights;
      Screen screen;
      PopcornPopper popper;

      public HomeTheaterFacade(Amplifier amp,
        Tuner tuner,
        DvdPlayer dvd,
        CdPlayer cd,
        Projector projector,
        Screen screen,
        PopcornPopper popper) {
          this.amp = amp;
          this.tuner = tuner;
          this.dvd = dvd;
          this.cd = cd;
          this.projector = projector;
          this.screen = screen;
        }

      public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...");
        popper.on();
        popper.pop();
        lights.dim(10);
        screen.down();
        projector.on();
        projector.wideScreenMode();
        amp.on();
        amp.setSurroundSound();
        amp.setVolume(5);
        dvd.on();
        dvd.play(movie);
      }

      public void endMovie() {
        System.out.println("Shutting movie theater down...");
        popper.off();
        lights.on();
        screen.up();
        projector.off();
        amp.off();
        dvd.stop();
        dvd.eject();
        dvd.off();
      }
    }
  ```

  Main File:

  ```java
    public class HomeTheaterTestDrive {
      public static void main(String[] args) {
        HomeThetaterFacade homeTheater =
          new HomeTheaterFacade(amp, tuner, dvd, cd, projector, screen, lights, popper);

        homeTheater.watchMovie("The Avengers");
        homeTheater.endMovie();
      }
    }
  ```

### Template Method

- Template Method defines the step of an algorithm and allows subclasses to provide the implementation for one or more steps.
- Example (Taken from Head-First Design Pattern pg. 281):

  - Steps to brew a cup of coffee:
    <ol>
      <li>Boil some water</li>
      <li>Brew coffee in boiling water</li>
      <li>Pour coffee in a cup</li>
      <li>Add sugar and milk</li>
    </ol>

    Steps to brew a cup of tea:
    <ol>
      <li>Boil some water</li>
      <li>Steep tea in boiling water</li>
      <li>Pour tea in a cup</li>
      <li>Add lemon</li>
    </ol>
    <br/>
    Notice that <b>both share the same algorithm</b>. <b>Step (1) & (3) can be abstracted</b> into the base class, while <b>step (2) & (4) aren't abstracted</b> although they are the same, they just apply to different beverages.
    <br/>
    Sample Code:

    ```java
      public abstract class CaffeineBeverage {
        final void prepareReceipe() {
          boilWater();
          brew();
          pourInCup();
          addCondiments();
        }

        abstract void brew();           // Step 2
        abstract void addCondiments();  // Step 4

        void boilWater() {
          System.out.println("Boiling some water...");
        }

        void pourInCup() {
          System.out.println("Pouring into a cup...");
        }
      }

      public class Tea extends CaffeineBeverage {
        public void brew() {
          System.out.println("Steeping the tea...");
        }
        public void addCondiments() {
          System.out.println("Adding lemon...");
        }
      }

      public class Coffee extends CaffeineBeverage {
        public void brew() {
          System.out.println("Dripping coffee through filter...");
        }
        public void addCondiments() {
          System.out.println("Adding sugar and milk...");
        }
      }
    ```

    Main File:

    ```java
      public static void main(String[] args) {
        Tea tea = new Tea();
        Coffee coffee = new Coffee();

        System.out.println("Making tea...");
        tea.prepareReceipe();

        System.out.println("Making coffee...");
        coffee.prepareReceipe();
      }
    ```

### Builder Pattern

- Builder pattern <b>separates the construction of a complex object from its representation so that the same construction process can create different representations</b>.
- Main difference between template method and builder pattern:
  <ul>
    <li><b>Template Method</b>: You create the sequence of steps and subclasses can make alterations</li>
    <li><b>Builder Pattern</b>: Client code sets the sequence and number of steps in the algorithm and swaps between the builders you provide to create various objects that embody that algorithm</li>
  </ul>
- Use the Builder design pattern when you want client code to have control over the construction process but want to be able to end up with different kinds of objects (each of which is built by a different type of builder).
- Example (Taken from Design Pattern for Dummies pg. 186):

  - To let the client build robots to perform various sequences of actions — starting, assembling, stopping, and so on -- I make the RobotBuilder interface support these methods: <b><i>addStart, addGetParts, addAssemble, addTest,</i></b> and <b><i>addStop</i></b>.

  - For example, <b><i>to make the robot under construction start, test, assemble, and then stop</b></i>, the client code only needs to call the builder’s <b><i>addStart, addTest, addAssemble, and addStop methods</b></i>, in that order.

  - <b>When the robot has been constructed</b>, the client code only needs to call the builder’s <b><i>getRobot</b></i> method to get the new robot. And the new robot object will support a go method that, when called, executes the sequence of actions it’s been constructed to run.

  - Because you can have multiple types of robot builders — those that build cookie robots or automotive robots, for example — I start by creating a RobotBuilder interface that all robot builders have to implement. This interface lists the methods that all robot builders must implement, from addStart to addStop, as well as the getRobot method:

    ```java
      public interface RobotBuilder {
        public void addStart();
        public void addGetParts();
        public void addAssemble();
        public void addTest();
        public void addStop();
        public RobotBuildable getRobot();
      }
    ```

  - All robots, for example: <code>CookieRobotBuilder</code> must implements <code>RobotBuilder</code> interface.
    The constructed robot is going to be a <code>CookieRobotBuildable</code> object, so we’ll need an object of that class in the builder (this is the object the builder will return as the fully constructed robot)
    The client code can add robot actions like <b>start, stop, test, assemble, getParts</b> , and so on in any order, and each action can be added any number of times. To keep track of the sequence of actions the client code wants to build into the current robot, I use an <code>ArrayList</code> object named <b>actions</b> in the builder.

    ```java
      import java.util.*;

      public class CookieRobotBuilder implements RobotBuilder {
        CookieRobotBuildable robot;
        ArrayList<Integer> actions;

        public CookieRobotBuilder() {
          robot = new CookieRobotBuildable();
          actions = new ArrayList<Integer>();
        }

        public void addStart() {
          actions.add(new Integer(1));
        }
        public void addGetParts() {
          actions.add(new Integer(2));
        }
        public void addAssemble() {
          actions.add(new Integer(3));
        }
        public void addTest() {
          actions.add(new Integer(4));
        }
        public void addStop() {
          actions.add(new Integer(5));
        }
        public RobotBuildable getRobot() {
          robot.loadActions(actions);
          return robot;
        }
      }
    ```

  - Each type of builder builds a different type of robot, and each robot is based on its own class, such as the <code>CookieRobotBuildable</code> or <code>Automotive RobotBuildable</code> class. All robots have to have a <code>go</code> method to make them execute their actions, so you might start with an interface, RobotBuildable, that makes sure that the <code>go</code> method is implemented in all robots.

    ```java
      public interface RobotBuildable {
        public void go();
      }
    ```

    Now all <code>Robot</code> classes will implement this interface.

    ```java
      import java.util.*;

      public class CookieRobotBuildable implements RobotBuilable {
        ArrayList<Integer> actions

        public CookieRobotBuildable() {}

        public loadActions(ArrayList<Integer> a) {
          actions = a;
        }

        public final void go() {
          Iterator it = actions.iterator();
          while (it.hasNext()) {
            switch((Integer) it.next()) {
              case 1:
                start(); break;
              case 2:
                getParts(); break;
              case 3:
                assemble(); break;
              case 4:
                test(); break;
              case 5:
                stop(); break;
            }
          }
        }

        public void start() {
          System.out.println("starting...");
        }
        public void getParts() {
          System.out.println("Getting flour and sugar...");
        }
        public void assemble() {
          System.out.println("Baking a cookie...");
        }
        public void test() {
          System.out.println("Crunching a cookie...");
        }
        public void stop() {
          System.out.println("Stopping...");
        }
      }
    ```

    <b>Testing the Robot Builder</b>

    ```java
      import java.io.*;

      pubic class TestRobotBuilder {
        public static void main(String args[]) {
          RobotBuilder builder;
          RobotBuildable robot;
          String response = "a";

          System.out.print("Do you want a cookie robot [c] or an automotive one [a]? ");
          BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

          try {
            response = reader.readLine();
          } catch(IOException e) {
            System.err.println("Error");
          }

          if (response.equals("c")) {
            builder = new CookieRootBuilder();
          } else {
            builder = new AutomotiveRobotBuilder();
          }

          // Start the construction process.
          builder.addStart();
          builder.addTest();
          builder.addAssemble();
          builder.addStop();

          robot = builder.getRobot();
          robot.go();
        }
      }
    ```

    - Cookie Robot:

    ```
      Do you want a cookie robot [c] or an automotive one [a]? c
      Starting...
      Crunching a cookie...
      Baking a cookie...
      Stopping...
    ```

    - Automotive Robot:

    ```
      Do you want a cookie robot [c] or an automotive one [a]? a
      Starting...
      Revving the engine...
      Installing the carburetor...
      Stopping...
    ```

### Iterator Pattern

- The Iterator pattern gives you a way of accessing the elements inside an object without having to know the internal details of that object.
- For example, Sun has introduced all kinds of collections in Java relatively recently, and these collections allow you to create iterators — special objects designed to give you access to the members of a collection — for easy access.
- The Iterator design pattern is especially important when the collection you’re creating is made up internally of separate subcollections, as when you’ve mixed hashes with array lists, for example.
- Example (taken from Head-First Design Pattern pg. 322):

  - Lou and Mel agree on the implementation of MenuItems, however Lou used ArrayList to hold his menu items while Mel used Array. Neither of them willing to change their implementations.
  - Implementation of <b>MenuItem</b>:

    ```java
    public class MenuItem {
      String name;
      String description;
      boolean vegetarian;
      double price;

      public MenuItem(String name,
                      String description,
                      boolean vegetarian,
                      double price)
      {
        // A MenuItem consists of a name, a description, a flag to indicate if them is vegetarian, and a price.
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
      }

      // Getter Methods
      public String getName() {
        return name;
      }
      public String getDescription() {
        return description;
      }
      public double getPrice() {
        return price;
      }
      public boolean isVegetarian() {
        return vegetarian;
      }
    }

    ```

  - <b>Lou's</b> Menu Implementations:

    ```java
    public class PancakeHouseMenu {
      ArrayList menuItems;

      public PancakeHouseMenu() {
        menuItems = new ArrayList();

        addItem("K&B's Pancake Breakfast",
                "Pancake with scrambled eggs, and toast",
                true,
                2.99);
        addItem("Regular Pancake Breakfast",
                "Pancake with fried eggs, sausage",
                false,
                2.99);
        addItem("Blueberry Pancakes",
                "Pancake made with fresh blueberries",
                true,
                3.49);
        addItem("Waffles",
                "Waffles, with your choice of blueberries or strawberries",
                true,
                3.59);
      }

      public void addItem(String name, String description,
                          boolean vegetarian, double price)
      {
        MenuItem menuItem = new MenuItem(name, description, vegetarian price);
        menuItems.add(menuItem);
      }

      public ArrayList getMenuItems() {
        return menuItems;
      }
    }
    ```

  - <b>Mel's</b> Menu Implementations:

    ```java
    public class DinnerMenu {
      static final int MAX_ITEMS = 6;
      int numberOfItems = 0;
      MenuItem[] menuItems;   // Mel use Array instead of ArrayList

      public DinnerMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        addItem("Vegetarian BLT",
                "(Fakin') Bacon with lettuce & tomato on whole wheat",
                true,
                2.99);
        addItem("BLT",
                "Bacon with lettuce & tomato on whole wheat",
                false,
                2.99);
        addItem("Soup of the day",
                "Soup of the day, with a side of potato salad",
                false,
                3.29);
        addItem("Hotdog",
                "A hot dog, with saurkraut, relish, onions, topped with cheese",
                false,
                3.05);
      }

      public void addItem(String name, String description
                          bool vegetarian, double price)
      {
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        if (numberOfItems >= MAX_ITEMS) {
          System.err.println("Sorry, menu is full! Can't add item to menu!");
        } else {
          menuItems[numberOfItems] = menuItem;
          numberOfItems = numberOfItems + 1;
        }
      }

      public MenuItem[] getMenuItems() {
        return menuItems;
      }
    }
    ```

  - Java-Enabled Waitress Specification:

    ```java
    printMenu()
      - prints every item on the menu

    printBreakFastMenu()
      - prints just breakfast items

    printLunchMenu()
      - prints just lunch items

    printVegetarianMenu()
      - prints all vegetarian menu items

    isItemVegetarian(name)
      - given the name of an item, returns true if the item is vegtarian, false otherwise
    ```

    Implementation <b>printMenu()</b> method:

    ```java
    (1) Retrieve menu items:
        PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();
        ArrayList breakfastItems = pancakeHouseMenu.getMenuItems();

        DinnerMenu dinnerMenu = new DinnerMenu();
        MenuItem[] lunchItems = dinnerMenu.getItems();

    (2) Iterate menu items:
        for (int i = 0; i < breakfastItems.size(); i ++) {
          MenuItem menuItem = (MenuItem) breakfastItems.get(i);
          System.out.println(menuItem.getName() + " ");
          System.out.println(menuItem.getPrice() + " ");
          System.out.println(menuItem.getDescription() + " ");
        }

        for (int i = 0; i < lunchItems.size(); i ++) {
          MenuItem menuItem = lunchItems[i];
          System.out.println(menuItem.getName() + " ");
          System.out.println(menuItem.getPrice() + " ");
          System.out.println(menuItem.getDescription() + " ");
        }

    (3) Implementing every other method in the Waitress is going to be a variation  of this theme.
        We're always going to need to get both menus and use two loops to iterate through their items.
        If another restaurant with a different implementation is acquired, then we'll have three loops.
    ```

  - <b>Solution</b>:
    This problem can be solved by using Iterator Pattern that relies on an interface called Iterator. Iterator interface may look like this:

    ```java
    public interface Iterator {
      boolean hasNext();
      Object next();
    }
    ```

    Concrete Iterator implementation that works for the Dinner menu:

    ```java
    public class DinnerMenuIterator implements Iterator {
      MenuItem[] menuItems;
      int position = 0;

      public DinnerMenuIterator(MenuItem[] items) {
        this.items = items;
      }

      public Object next() {
        MenuItem menuItem = items[position];
        position = position + 1;
        return menuItem;
      }

      public boolean hasNext() {
        if (position >= items.length || items[position] == null) {
          return false;
        }
        return true;
      }
    }
    ```

    Reworking the Dinner Menu with Iterator:

    ```java
    public class DinnerMenu {
      static final int MAX_ITEMS = 6;
      int numberOfItems = 0;
      MenuItem[] menuItems;

      // constructor here
      // addItem here

      /*
      We're not going to use this method anymore
      public MenuItem[] getMenuItems {
        return menuItem;
      }
      */

      public Iterator createIterator {
        return new DinnerMenuIterator(menuItems);
      }
    }
    ```

    Fixing up the Waitress Code

    ```java
    public class Waitress {
      PancakeHouseMenu pancakeHouseMenu;
      DinnerMenu dinnerMenu;

      public Waitress(PancakeHouseMenu pancakeHouseMenu, DinnerMenu dinnerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinnerMenu = dinnerMenu;
      }

      public void printMenu() {
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        Iterator dinnerIterator = dinnerMenu.createIterator();
        System.out.println("MENU\n----\nBREAKFAST");
        printMenu(pancakeIterator);
        System.out.println("\nLUNCH");
        printMenu(dinnerIterator);
      }

      private void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
          MenuItem menuItem = (MenuItem) iterator.next();
          System.out.println(menuItem.getName() + ", ");
          System.out.println(menuItem.getPrice() + " -- ");
          System.out.println(menuItem.getDescription());
        }
      }
    }
    ```

    Main File:

    ```java
    public class MenuTestDrive {
      public static void main(String[] args) {
        PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();
        DinnerMenu dinnerMenu = new DinnerMenu();

        Waitress waitress = new Waitress(pancakeHouseMenu, dinnerMenu);
        waitress.printMenu();
      }
    }

    ```

  - Some improvements that can be made:
    Change it to `java.util.Iterator` in PancakeHouseMenu

    ```java
    public Iterator createIterator() {
      return menuItems.iterator();
    }
    ```

    Also, we need to make the changes to allow the DinnerMenu to work with `java.util.Iterator`

    ```java
    import java.util.Iterator;

    public class DinnerMenuIterator implements Iterator {
      MenuItem[] list;
      int position = 0;

      public DinnerMenuIterator(MenuItem[] list) {
        this.list = list;
      }

      public Object next() {
        // Implementation here
      }

      public boolean hasNext() {
        // Implementation here
      }

      public void remove() {
        if (position <= 0) {
          throw new IllegalStateException(
            "You can't remove an item until you've done at least one next()"
          );
        }
        if (list[position - 1] != null) {
          for (int i = poisition - 1; i < (list.length - 1); i ++) {
            list[i] = list[i + 1];
          }
          list[list.length - 1] = null;
        }
      }
    }
    ```

    We just need to give the Menus a common interface and rework the Waitress a little. We might want to add few more methods, like `addItem()`, but for now we will let the chefs control their menus by keeping that method out of the public interface

    ```java
    public interface Menu() {
      public Iterator createIterator();
    }
    ```

    Then, we need to add an `implements Menu` to both PancakeHouseMenu and DinnerMenu class definitions.

    ```java
    import java.util.Iterator;    // Now the waitress uses the java.util.Iterator as well

    public class Waitress {
      Menu pancakeHouseMenu;
      Menu dinnerMenu;

      // Replace concrete Menu classes with the Menu Interface
      public Waitress(Menu pancakeHouseMenu, Menu dinnerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinnerMenu = dinnerMenu;
      }

      public void printMenu() {
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        Iterator dinnerIterator = dinnerMenu.createIterator();
        System.out.println("MENU\n----\nBREAKFAST");
        printMenu(pancakeIterator);
        System.out.println("\nLUNCH");
        printMenu(dinnerIterator);
      }

      private void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
          MenuItem menuItem = (MenuItem) iterator.next();
          System.out.println(menuItem.getName() + ", ");
          System.out.println(menuItem.getPrice() + " -- ");
          System.out.println(menuItem.getDescription());
        }
      }
    }
    ```

    <b>What does this get us?</b>
    <ul>
      <li>The PancakeHouseMenu and DinnerMenu classes implements an interface, Menu. Waitress can refer to each menu object using the interface rather than the concrete class. So, we're reducing the dependency between the Waitress and the concrete classes by <i><code>"programming to an interface, not an implementation"</code></i></li>
      <li>The new Menu interface has one method, <code>createIterator()</code> that is implemented by PancakeHouseMenu and DinnerMenu. Each menu class assumes the responsibility of creating a concrete Iterator that is appropriate for its internal implementation of the menu items.
      <br/>
      This solves the problem of the waitress depending on the implementation of the MenuItems.</li>
    </ul>

  - <b>What if the menus are mixed with another menu that use different data structure? (i.e. Hash Table)</b>

    - Example of CafeMenu implementation:

      ```java
      public class CafeMenu {
        Hashtable menuItems = new Hashtable();

        public CafeMenu() {
          addItem("Veggie Burger and Air Fries",
                  "Veggie burger on a whole wheat bun, lettuce, tomato, and fries",
                  false 3.99);
          addItem("Soup of the day",
                  "A cup of the soup of the day, with a side salad",
                  false 3.69);
          addItem("Burrito",
                  "A large burrito, with whole pinto beans, salsa, guacamole",
                  false 4.29);
        }

        public void addItem(String name, String description,
                            boolean vegetarian, double price)
        {
          MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
          // Key-value pair
          menuItems.put(menuItem.getName(), menuItem);
        }

        public Hashtable getItems() {
          return menuItems;
        }
      }
      ```

      Reworking the Cafe Menu

      ```java
      public class CafeMenu implements Menu {
        Hashtable menuItems = new Hashtable();

        public CafeMenu() {
          // constructor code here
        }

        public void addItem(String name, String description,
                            boolean vegetarian, double price)
        {
          MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
          menuItems.put(menuItem.getName(), menuItem);
        }

        /*
        Just like before, we get ride of getItems() so we don't expose the implementation of menuItems to the waitress
        public Hashtable getItems() {
          return menuItems;
        }
        */

        // Instead, we implement the createIterator() method.
        public Iterator createIterator() {
          return menuItems.iterator();
        }
      }
      ```

      Adding the Cafe Menu to the Waitress

      ```java
      public class Waitress {
        Menu pancakeHouseMenu;
        Menu dinnerMenu;
        Menu cafeMenu;

        public Waitress(Menu pancakeHouseMenu, Menu dinnerMenu, Menu cafeMenu) {
          this.pancakeHouseMenu = pancakeHouseMenu;
          this.dinnerMenu = dinnerMenu;
          this.cafeMenu = cafeMenu;
        }

        public void printMenu() {
          Iterator pancakeIterator = pancakeHouseMenu.createIterator();
          Iterator dinnerIterator = dinnerMenu.createIterator();
          Iterator cafeIterator = cafeMenu.createIterator();
          System.out.println("MENU\n---\nBREAKFAST");
          printMenu(pancakeIterator);
          System.out.println("\nLUNCH");
          printMenu(dinnerIterator);
          System.out.println("\nDINNER");
          printMenu(cafeIterator);
        }

        private void printMenu(Iterator iterator) {
          while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName() + ", ");
            System.out.println(menuItem.getPrice() + " -- ");
            System.out.println(menuItem.getDescription());
          }
        }
      }
      ```

      And finally, the main file:

      ```java
      public class MenuTestDrive {
        public static void main(String args[]) {
          PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();
          DinnerMenu dinnerMenu = new DinnerMenu();
          CafeMenu cafeMenu = new CafeMenu();

          Waitress waitress = new Waitress(pancakeHouseMenu, dinnerMenu, cafeMenu);
          waitress.printMenu();
        }
      }
      ```

### Composite Pattern

- Composite pattern allows us to build structures of objects in the form of trees that contain both compositions of objects and individual objects as nodes.
- Using a composite structure, we can apply the same operations over both composites and individual objects. In other words, in most cases we <b>ignore</b> the difference between compositions of objects and individual objects.
- Example (Taken from Head-First Design Pattern pg. 366):

  - To apply composite pattern to our menus, we need to create a component interface which acts as the common interface for both menus and menu items and allows us to treate them uniformly.
    In other words, we can <b>call the same method</b> on menus or menu items.
  - Implementation of <b>MenuComponent</b>:

    ```java
    public abstract class MenuComponent {
      public void add(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
      }
      public void remove(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
      }
      public void getChild(int i) {
        throw new UnsupportedOperationException();
      }

      public void getName() {
        throw new UnsupportedOperationException();
      }
      public void getDescription() {
        throw new UnsupportedOperationException();
      }
      public void getPrice() {
        throw new UnsupportedOperationException();
      }
      public void isVegetarian() {
        throw new UnsupportedOperationException();
      }

      public void print() {
        throw new UnsupportedOperationException();
      }
    }
    ```

    Implementation of <b>MenuItem</b>:

    ```java
    public class MenuItem extends MenuComponent {
      String name;
      String description;
      boolean vegetarian;
      double price;

      public MenuItem(String name, String description,
                      boolean vegetarian, double price)
      {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
      }

      public String getName() {
        return name;
      }
      public String getDescription() {
        return description;
      }
      public String getPrice() {
        return price;
      }
      public String isVegetarian() {
        return vegetarian;
      }

      public void print() {
        System.out.print(" " + getName());
        if (isVegetarian()) {
          System.our.print("(v)");
        }
        System.out.println(", " + getPrice());
        System.out.println("   -- " + getDescription());
      }
    }
    ```

    Implementation of <b>Composite Menu</b>:

    ```java
    public class Menu extends MenuComponent {
      ArrayList menuComponents = new ArrayList();
      String name;
      String description;

      public Menu(String name, String description) {
        this.name = name;
        this.description = description;
      }

      public void add(MenuComponent menuComponent) {
        menuComponents.add(menuComponent);
      }
      public void remove(MenuComponent menuComponent) {
        menuComponents.remove(menuComponent);
      }
      public MenuComponent getChild(int i) {
        return (MenuComponent) menuComponents.get(i);
      }
      public String getName() {
        return name;
      }
      public String getDescription() {
        return description;
      }
      public void print() {
        System.out.print("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("---------------");

        Iterator iterator = menuComponents.iterator();
        // We use iterator to iterate through all of the Menu's components... those could be other Menus or MenuItems.
        while (iterator.hasNext()) {
          MenuComponent menuComponent = (MenuComponent) iterator.next();
          menuComponent.print();
        }
      }
    }
    ```

    Waitress Implementation:

    ```java
    public class Waitress {
      MenuComponent allMenus;

      public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
      }

      public void printMenu() {
        allMenus.print();
      }
    }
    ```

    Main File:

    ```java
    public class MainTestDrive {
      public static void main(String[] args) {
        MenuComponent pancakeHouseMenu = new Menu("PANCAKE HOUSE MENU", "Breakfast");
        MenuComponent dinnerMenu = new Menu("DINNER MENU", "Lunch");
        MenuComponent pancakeHouseMenu = new Menu("CAFE MENU", "Dinner");
        MenuComponent dessertMenu = new Menu("DESSERT MENU", "Dessert of course!");

        allMenus.add(pancakeHouseMenu);
        allMenus.add(dinnerMenu);
        allMenus.add(cafeMenu);

        dinnerMenu.add(new MenuItem(
          "Pasta",
          "Spaghetti with Marinara Sauce, and a slice of sourdough bread",
          true,
          3.89,
        ));
        dinnerMenu.add(dessertMenu);

        dessertMenu.add(new MenuItem(
          "Apple Pie",
          "Apple pie with a flakey crust, topped with vanilla icecream",
          true,
          1.59,
        ));
        // Add more menu items here

        Waitress waitress = new Waitress(allMenus);
        waitress.printMenu();
      }
    }
    ```

- Some changes that can be made:

  - This violates the "One class, one responsibility" principle since composite pattern manages a hierarchy <b>AND</b> it performs operations related to Menus.
  - To solve this, we can implement a Composite iterator <code>createIterator()</code> method in every component.
  - <b>Menu</b> and <b>MenuItem</b> classes:

    ```java
    public class Menu extends MenuComponent {
      // other code here doesn't change

      public Iterator createIterator() {
        return new CompositeIterator(menuComponents.iterator());
      }
    }

    public class MenuItem extends MenuComponent {
      // other code here doesn't change

      public Iterator createIterator() {
        return new NullIterator();
      }
    }
    ```

    Iterators implementation:

    - Composite Iterator

      ```java
      import java.util.*;

      public class CompositeIterator implements Iterator {
        Stack stack = new Stack();

        public CompositeIterator(Iterator iterator) {
          stack.push(iterator);
        }

        public Object next() {
          if (hasNext()) {
            Iterator iterator = (Iterator) stack.peek();
            MenuComponent component = (MenuComponent) iterator.next();
            if (component instanceof Menu) {
              stack.push(component.createIterator());
            }
            return component;
          }
          return null;
        }

        public boolean hasNext() {
          if (stack.empty()) {
            return false;
          }

          Iterator iterator = (Iterator) stack.peek();
          if (!iterator.hasNext()) {
            stack.pop();
            return hasNext();
          }
          return true;
        }

        // Not supporting remove, only traversal.
        public void remove() {
          throw new UnsupportedOperationException();
        }
      }
      ```

    - Null Iterator

      ```java
      import java.util.Iterator;

      public class NullIterator implements Iterator {
        public Object next() {
          return null;
        }

        public boolean hasNext() {
          return false;
        }

        public void remove() {
          throw new UnsupportedOperationException();
        }
      }
      ```

  - Waitress class revisited

    ```java
    public class Waitress {
      MenuCompoent allMenus;

      public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
      }

      public void printMenu() {
        allMenus.print();
      }

      public void printVegetarianMenu() {
        Iterator iterator = allMenus.createIterator();
        System.out.println("\nVEGETARIAN MENU\n----");

        // Iterate through every element of the composite
        while (iterator.hasNext()) {
          MenuComponent menuComponent = (MenuComponent) iterator.next();

          try {
            if (menuComponent.isVegetarian()) {
              menuComponent.print();
            }
          } catch(UnsupportedOperationException e) {}
        }
      }
    }
    ```

### State Pattern

- State Pattern allows an oject to alter its behaviour when its internal state changes. The object will appear to change its class.
- Preparations (Taken from Head-First Design Pattern pg. 395):

  - List of states:
    <ol>
      <li>Sold Out</li>
      <li>No Quarter</li>
      <li>Has Quarter</li>
      <li>Sold</li>
    </ol>
  - Create an instance variable to holdthe curent state and the values:

    ```java
    final static int SOLD_OUT = 0;
    final static int NO_QUARTER = 1;
    final static int HAS_QUARTER = 2;
    final static int SOLD = 3;

    int state = SOLD_OUT;
    ```

  - List of actions:
    <ol>
      <li>Inserts Quarter</li>
      <li>Ejects Quarter</li>
      <li>Turns Crank</li>
      <li>Dispense</li>
    </ol>

  - Writing the code:

    ```java
    public class GumballMachine {
      final static int SOLD_OUT = 0;
      final static int NO_QUARTER = 1;
      final static int HAS_QUARTER = 2;
      final static int SOLD = 3;

      int state = SOLD_OUT;
      int count = 0;

      public GumballMachine(int count) {
        this.count = count;
        if (count > 0) {
          state = NO_QUARTER;
        }
      }

      public void insertQuarter() {
        if (state == HAS_QUARTER) {
          System.out..println("You can't insert another quarter");
        } else if (state == NO_QUARTER) {
          System.out.println("You inserted a quarter");
        } else if (state == SOLD_OUT) {
          System.our.println("You can't insert a quarter, the machine is sold out");
        } else if (state == SOLD) {
          System.out.println("Please wait, we're already giving you a gumball");
        }
      }

      public void ejectQuarter() {
        if (state == HAS_QUARTER) {
          System.out..println("Quarter returned");
          state = NO_QUARTER;
        } else if (state == NO_QUARTER) {
          System.out.println("You haven't inserted a quarter");
        } else if (state == SOLD) {
          System.our.println("Sorry, you already turned the crank");
        } else if (state == SOLD_OUT) {
          System.out.println("You can't eject, you haven't inserted a quarter yet");
        }
      }

      public void turnCrank() {
        if (state == SOLD) {
          System.out..println("Turning twice doesn't get you another gumball");
        } else if (state == NO_QUARTER) {
          System.out.println("You turned but there's no quarter");
        } else if (state == SOLD_OUT) {
          System.out.println("You turned, but there are no gumballs");
        } else if (state == HAS_QUARTER) {
          System.our.println("You turned...");
          state = SOLD;
          dispense();
        }
      }

      public void dispense() {
        if (state == SOLD) {
          System.out.println("A gumball comes rolling out the slot");
          count --;
          if (count == 0) {
            System.out.println("Oops, outs of gumballs!");
            state = SOLD_OUT;
          } else {
            state = NO_QUARTER;
          }
        } else if (state == NO_QUARTER) {
          System.out..println("You need to pay first");
        } else if (state == SOLD_OUT) {
          System.out.println("No gumball dispensed");
        } else if (state == HAS_QUARTER) {
          System.out.println("No gumball dispensed");
        }
      }
    }
    ```

    Main File:

    ```java
    public class GumballMachineTestDrive {
      public static void main(String[] args) {
        // Load it up with 5 gumball total
        GumballMachine ghmballMachine = new GumballMachine(5);

        System.out.println(gumballMachine);

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        System.out.println(gumbalMachine);

        gumballMachine.insertQuarter();
        gumballMachine.ejectQuarter();
        gumballMachine.turnCrank();

        System.out.println(gumballMachine);

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.ejectQuarter();

        System.out.println(gumballMachine);

        gumballMachine.insertQuarter();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        System.out.println(gumballMachine);
      }
    }
    ```

  - Messy STATE of things:

    ```java
    final static int SOLD_OUT = 0;
    final static int NO_QUARTER = 1;
    final static int HAS_QUARTER = 2;
    final static int SOLD = 3;
    /*
      First, you'd have to add a new WINNER state here. That isn't too bad
      But then, you'd have to add a new conditional in every single method to handle the WINNER state;
      That's a lot of code to modify
    */
    public void insertQuarter() {
      // insert quarter code here
    }
    public void ejectQuarter() {
      // eject quarter code here
    }
    public void turnCrank() {
      // turn crank code here
    }

    public void dispense() {
      // dispense code here
    }
    ```

- The new design

  - Here is the design:
    <ol>
    <li>First, we're going to define a State interface that contains a method for every action in the Gumball Machine</li>
    <li>Then, we're going to implement a State class for every state of the machine. These classes will be responsible for the behaviour of the machine when it is in the corresponding state.</li>
    <li>Finally, we're going to get rid of all our conditional code and instead delegate to the state class to do the work for us.</li>
    </ol>
  - Implementing states:
    List of states:
    <ul>
    <li>NoQuarterState</li>
    <li>SoldOutState</li>
    <li>SoldState</li>
    <li>HasQuarterState</li>
    </ul>
    <br/>
    <b>NoQuarterState</b>

    ```java
    public class NoQuarterState implements State {
      GumballMachine gumballMachine;

      public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
      }

      public void insertQuarter() {
        System.out.println("You inserted a quarter");
        gumballMachine.setState(gumballMachine.getHasQuarterState());
      }

      public void ejectQuarter() {
        System.out.println("You haven't inserted a quarter");
      }

      public void turnCrank() {
        System.out.println("You turned, but there's no quarter");
      }

      public void dispense() {
        System.out.println("You need to pay first");
      }
    }
    ```

    <b>HasQuarterState</b>

    ```java
    public class HasQuarterState implements State {
      GumballMachine gumballMachine;

      public HasQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
      }

      public void insertQuarter() {
        System.out.println("You can't insert another quarter");
      }

      public void ejectQuarter() {
        System.out.println("Quarter returned");
        gumballMachine.setState(gumballMachine.getNoQuarterState());
      }

      public void turnCrank() {
        System.out.println("You turned...");
        gumballMachine.setState(gumballMachine.getSoldState());
      }

      public void dispense() {
        System.out.println("No gumball dispensed");
      }
    }
    ```

    <b>SoldState</b>

    ```java
    public class SoldState implements State {
      GumballMachine gumballMachine;

      public SoldState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
      }

      public void insertQuarter() {
        System.out.println("Please wait, we're already giving you a gumball");
      }

      public void ejectQuarter() {
        System.out.println("Sorry, you already turned the crank");
      }

      public void turnCrank() {
        System.out.println("Turning twice doesn't get you another gumball!");
      }

      public void dispense() {
        gumballMachine.releaseBall();
        if (gumballMachine.getCount() > 0) {
          gumballMachine.setState(gumballMachine.getNoQuarterState());
        } else {
          System.out.println("Oops, out of gumballs!");
          gumballMachine.setState(gumballMachine.getSoldOutState());
        }
      }
    }
    ```

    <b>SoldOutState</b>

    ```java
    public class SoldOutState implements State {
      GumballMachine gumballMachine;

      public SoldOutState(GumballMachine gumballMachine) {
          this.gumballMachine = gumballMachine;
      }

      public void insertQuarter() {
        System.out.println("You can't insert a quarter, the machine is sold out");
      }

      public void ejectQuarter() {
        System.out.println("You can't eject, you haven't inserted a quarter yet");
      }

      public void turnCrank() {
        System.out.println("You turned, but there are no gumballs");
      }

      public void dispense() {
        System.out.println("No gumball dispensed");
      }

      public void refill() {
        gumballMachine.setState(gumballMachine.getNoQuarterState());
      }
    }
    ```

    <b>Gumball Machine</b>

    ```java
    public class GumballMachine {

      State soldOutState;
      State noQuarterState;
      State hasQuarterState;
      State soldState;

      State state;
      int count = 0;

      public GumballMachine(int numberGumballs) {
        soldOutState = new SoldOutState(this);
        noQuarterState = new NoQuarterState(this);
        hasQuarterState = new HasQuarterState(this);
        soldState = new SoldState(this);

        this.count = numberGumballs;
        if (numberGumballs > 0) {
          state = noQuarterState;
        } else {
          state = soldOutState;
        }
      }

      public void insertQuarter() {
        state.insertQuarter();
      }

      public void ejectQuarter() {
        state.ejectQuarter();
      }

      public void turnCrank() {
        state.turnCrank();
        state.dispense();
      }

      void releaseBall() {
        System.out.println("A gumball comes rolling out the slot...");
        if (count != 0) {
          count = count - 1;
        }
      }

    ```

    Main File:

    ```java
    public class GumballMachineTestDrive {
      public static void main(String[] args) {
        GumballMachine gumballMachine = new GumballMachine(2);

        System.out.println(gumballMachine);

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        System.out.println(gumballMachine);

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        gumballMachine.refill(5);
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        System.out.println(gumballMachine);
      }
    }
    ```

- Suppose there is a new state added:
  <b>Winner State</b>

  ```java
  public class WinnerState implements State {
    GumballMachine gumballMachine;

    // instance variables and constructor

    // insertQuarter error message

    // ejectQuarter error message

    // turnCrank error message

    public void dispense() {
      System.out.println("YOU'RE A WINNER! You got two gumballs for your quarter");
      gumballMachine.releaseBall();
      if (gumballMachine.getCount() == 0) {
        gumballMachine.setState(gumballMachine.getSoldOutState());
        return;
      }

      gumballMachine.releaseBall();
      if (gumballMachine.getCount() > 0) {
        gumballMachine.setState(gumballMachine.getNoQuarterState());
      } else {
        System.out.println("Oops, out of gumballs!");
        gumballMachine.setState(gumballMachine.getSoldOutState());
      }
    }
  }
  ```

  <b>GumballMachine</b>

  ```java
  public class GumballMachine {
    State soldOutState;
    State noQuarterState;
    State hasQuarterState;
    State soldState;
    State winnerState;    // Add new state here

    State state = soldOutState;
    int count = 0;

    // methods here
  }
  ```

  <br/>

  From here, we've just got one more change to make: we need to implement the random chance game and add a transition to the <b>WinnerState</b>. We're going to add both to the <b>HasQuarterState</b> since that is where the customer turns the crank:
  <br/>
  <b>HasQuarterState</b>

  ```java
  public class HasQuarterState implements State {
    // We add a random number generator
    Random randomWinner = new Random(System.currentTimeMillis());
    GumballMachine gumballMachine;

    public HasQuarterState(GumballMachine gumballMachine) {
      this.gumballMachine = gumballMachine;
    }

    public void insertQuarter() {
      System.out.println("You can't insert another quarter");
    }

    public void ejectQuarter() {
      System.out.println("Quarter returned");
      gumballMachine.setState(gumballMachine.getNoQuarterState());
    }

    public void turnCrank() {
      System.out.println("You turned...");

      // Here, we determine if this customer won or not
      int winner = randomWinner.nextInt(10);
      if ((winner == 0) && (gumballMachine.getCount() > 1)) {
        gumballMachine.setState(gumballMachine.getWinnerState());
      } else {
        gumballMachine.setState(gumballMachine.getSoldState());
      }
    }

    public void dispense() {
      System.out.println("No gumball dispensed");
    }
  }
  ```

  Here, we can see that it is pretty simple to implement. We just added a new state to the GumballMachine and then implemented it.

### Proxy Pattern
