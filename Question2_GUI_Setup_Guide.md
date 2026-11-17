# Question 2: Building the Movie Tickets GUI in NetBeans (Java with Maven)

This guide walks through Question 2 from an empty NetBeans window to a finished, tested application. The whole form is built with the NetBeans GUI Builder (drag and drop in Design view), so none of the layout code is typed by hand. The only code you write yourself is the logic classes, the menu event handlers and the unit tests.

The finished project is also in the zip as `MovieTicketsApp`. You can open it to compare, but if you have to show that you built the GUI yourself, follow these steps in a new project.

---

## What you need

- Apache NetBeans (version 20 or newer works best)
- JDK 17 or newer (the project targets Java 17, so JDK 17, 21 or 25 all work)

---

## Step 1: Create the Maven project

1. Open NetBeans and go to **File > New Project**.
2. Under **Categories** choose **Java with Maven**, under **Projects** choose **Java Application**, then click **Next**.
3. Fill in:
   - **Project Name:** `MovieTicketsApp`
   - **Group Id:** `com.mycompany`
   - **Package:** `com.mycompany.movieticketsapp` (NetBeans fills this in for you)
4. Click **Finish**.

NetBeans creates the project with a starter class called `MovieTicketsApp.java` under **Source Packages > com.mycompany.movieticketsapp**. You won't need it, because the form will have its own `main` method. Right-click it and choose **Delete**.

---

## Step 2: Add JUnit 5 to the pom.xml

1. In the **Projects** window, expand **Project Files** and open **pom.xml**.
2. Inside `<project>`, after the `</properties>` tag, add this block. If NetBeans already added a `<dependencies>` section, put just the `<dependency>` part inside it.

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

3. Save (**Ctrl+S**). NetBeans downloads JUnit the next time you build. If it doesn't, right-click the project and choose **Build** (or **Clean and Build**).

---

## Step 3: Create the logic classes (Q.2.7 and Q.2.8)

These three files hold the calculation and the validation. They're plain Java with no GUI.

### 3.1 The interface

1. Right-click the package **com.mycompany.movieticketsapp** and choose **New > Java Interface...**
2. Name it `IMovieTickets` and click **Finish**.
3. Replace the contents with:

```java
package com.mycompany.movieticketsapp;

public interface IMovieTickets {

    double CalculateTotalTicketPrice(int numberOfTickets, double ticketPrice);

    boolean ValidateData(MovieTicketData movieTicketData);
}
```

It will show a red error on `MovieTicketData` until the next class exists. That's expected.

### 3.2 The data class

Right-click the package, choose **New > Java Class...**, name it `MovieTicketData` and add:

```java
package com.mycompany.movieticketsapp;

public class MovieTicketData {

    private String movieName;
    private int numberOfTickets;
    private double ticketPrice;

    public MovieTicketData(String movieName, int numberOfTickets, double ticketPrice) {
        this.movieName = movieName;
        this.numberOfTickets = numberOfTickets;
        this.ticketPrice = ticketPrice;
    }

    public String getMovieName() { return movieName; }
    public void setMovieName(String movieName) { this.movieName = movieName; }

    public int getNumberOfTickets() { return numberOfTickets; }
    public void setNumberOfTickets(int numberOfTickets) { this.numberOfTickets = numberOfTickets; }

    public double getTicketPrice() { return ticketPrice; }
    public void setTicketPrice(double ticketPrice) { this.ticketPrice = ticketPrice; }
}
```

Tip: you can type only the three fields and the constructor, then right-click inside the class and choose **Insert Code... > Getter and Setter...** to generate the rest.

### 3.3 The MovieTickets class

Create another Java class called `MovieTickets`:

```java
package com.mycompany.movieticketsapp;

public class MovieTickets implements IMovieTickets {

    public static final double VAT_RATE = 0.14;

    @Override
    public double CalculateTotalTicketPrice(int numberOfTickets, double ticketPrice) {
        double subtotal = numberOfTickets * ticketPrice;
        double total = subtotal + (subtotal * VAT_RATE);
        return Math.round(total * 100.0) / 100.0;   // 3 x R100 gives R342.0
    }

    @Override
    public boolean ValidateData(MovieTicketData movieTicketData) {
        return getValidationMessage(movieTicketData).isEmpty();
    }

    // Returns "" when valid, otherwise a message the form can show the user
    public String getValidationMessage(MovieTicketData movieTicketData) {
        if (movieTicketData == null) {
            return "No movie ticket data was captured.";
        }
        if (movieTicketData.getMovieName() == null || movieTicketData.getMovieName().trim().isEmpty()) {
            return "Please select a movie.";
        }
        if (movieTicketData.getTicketPrice() <= 0) {
            return "The ticket price must be greater than zero.";
        }
        if (movieTicketData.getNumberOfTickets() <= 0) {
            return "The number of tickets must be greater than zero.";
        }
        return "";
    }
}
```

Why round? `300 * 1.14` in Java's `double` arithmetic isn't exactly 342. Rounding to two decimals makes the report show `R 342.0`, just like the sample screenshot.

---

## Step 4: Create the JFrame Form

1. Right-click the package **com.mycompany.movieticketsapp** and choose **New > JFrame Form...**
   - If you don't see it in the menu, choose **New > Other...**, then **Swing GUI Forms > JFrame Form**.
2. Name it `MovieTicketsForm` and click **Finish**.

NetBeans opens the form in **Design** view. Get to know these windows before you start dragging:

| Window | What it's for | If it's missing |
|---|---|---|
| **Palette** (right side) | The components you drag onto the form | **Window > IDE Tools > Palette** or **Ctrl+Shift+8** |
| **Properties** (right side, under Palette) | Text, name, editable and so on for the selected component | **Window > IDE Tools > Properties** or **Ctrl+Shift+7** |
| **Navigator** (bottom left) | Tree of everything on the form, which makes selecting things easy | **Window > Navigator** or **Ctrl+7** |

The **Source** and **Design** buttons at the top of the editor switch between the drag-and-drop view and the code. The grey code in Source view is generated by the GUI Builder and is locked. That's normal, and it's what proves the layout came from the designer.

---

## Step 5: Set up the frame itself

1. Click an empty area of the form, or click **JFrame** at the top of the **Navigator** tree.
2. In **Properties**:
   - **title:** `MOVIE TICKETS`
   - **defaultCloseOperation:** `EXIT_ON_CLOSE` (this is usually already set)
3. Drag the bottom-right corner of the form to make it about 430 x 370 so there's room to work. You can tidy the size at the end.

---

## Step 6: Drag the components onto the form (Q.2.1)

The form uses the default **Free Design** layout. When you drag something near another component, blue dotted guide lines appear. Let them snap so the labels line up on the left and the input boxes line up on the right.

### 6.1 Drag them on

From the **Palette > Swing Controls** section, drag these onto the form from top to bottom:

1. **Label**, top left. Then a **Combo Box** to its right on the same row (wait for the baseline guide to appear).
2. **Label** below the first, with a **Text Field** to its right.
3. **Label** below that, with a **Text Field** to its right.
4. **Label** below that, on its own row.
5. **Text Area** below the last label. NetBeans automatically wraps it in a **JScrollPane**. That's what you want, because the report can scroll if it's long.

### 6.2 Set the text and the variable names

For each component:

- **Change the text:** select it and edit **text** in **Properties**, or right-click and choose **Edit Text**.
- **Rename the variable:** right-click and choose **Change Variable Name...** Good names make the event code much easier to read.

| Component | Text (Properties > text) | Variable name |
|---|---|---|
| Label 1 | `MOVIE:` | `lblMovie` |
| Combo Box | (see 6.3) | `cmbMovie` |
| Label 2 | `NUMBER OF TICKETS:` | `lblNumberOfTickets` |
| Text Field 1 | delete `jTextField1` so it's empty | `txtNumberOfTickets` |
| Label 3 | `TICKET PRICE:` | `lblTicketPrice` |
| Text Field 2 | delete `jTextField2` so it's empty | `txtTicketPrice` |
| Label 4 | `TICKET REPORT:` | `lblTicketReport` |
| Text Area | (leave empty) | `txaReport` |

Once a text field is empty it shrinks. Drag its right edge back out so it's about as wide as the combo box. Do the same for the other text field so all three line up.

### 6.3 Fill the combo box with the movies

1. Select `cmbMovie`.
2. In **Properties**, find **model** and click the **...** button next to it.
3. Delete `Item 1` to `Item 4` and type these, one per line:
   ```
   Napoleon
   Oppenheimer
   Damsel
   ```
4. Click **OK**.
5. Optional but tidy: in **Properties**, open the **Code** tab and check that **Type Parameters** says `<String>`. Newer NetBeans versions set this by default.

### 6.4 Make the text area read only

1. Select `txaReport` (click inside the text area, or pick it in the **Navigator** under **jScrollPane1**).
2. In **Properties**, **untick editable**.

The user can still select and copy the report, but they can't type in it. That's the "read only" requirement in Q.2.1.

### 6.5 Tidy up

- Drag the scroll pane's right edge so it lines up with the right edge of the text fields, and make it about five lines tall.
- Click the **Preview Design** button (the small eye icon in the form toolbar) to see what the window will look like when it runs.

---

## Step 7: Add the menu system (Q.2.2)

### 7.1 The menu bar

1. In the **Palette**, scroll to **Swing Menus**.
2. Drag a **Menu Bar** onto the very top of the form and drop it when the top edge highlights.

NetBeans adds the bar with two menus already in it: **File** and **Edit**.

### 7.2 Rename the menus

1. In the Navigator, right-click **jMenuBar1** and choose **Change Variable Name...** Name it `menuBar`.
2. Right-click the **File** menu (on the form or in the Navigator), choose **Change Variable Name...** and name it `mnuFile`. Its text is already `File`.
3. Select the **Edit** menu:
   - change its **text** to `Tools` (double-click it on the form, or use **Properties > text**)
   - change its variable name to `mnuTools`

The question asks for a **Tools** menu, so use that even though the sample screenshot shows "Edit".

### 7.3 Add the menu items

1. From **Swing Menus**, drag a **Menu Item** and drop it onto the **File** menu on the form. The menu drops open when you hover over it.
   - Alternative: in the Navigator, right-click **mnuFile** and choose **Add From Palette > Menu Item**.
2. Set its **text** to `Exit` and its variable name to `mnuExit`.
3. Add two **Menu Items** to the **Tools** menu the same way:
   - **text** `Process`, variable `mnuProcess`
   - **text** `Clear`, variable `mnuClear`

Your **Navigator** should now look like this:

```
[JFrame]
├── menuBar
│   ├── mnuFile
│   │   └── mnuExit
│   └── mnuTools
│       ├── mnuProcess
│       └── mnuClear
├── lblMovie
├── cmbMovie
├── lblNumberOfTickets
├── txtNumberOfTickets
├── lblTicketPrice
├── txtTicketPrice
├── lblTicketReport
└── jScrollPane1
    └── txaReport
```

---

## Step 8: Hook up the events

Every menu item needs an **actionPerformed** event. NetBeans writes the method signature and wires up the listener for you. You only fill in the body.

To create an event handler for a menu item:

1. In the **Navigator**, right-click the menu item (for example `mnuExit`).
2. Choose **Events > Action > actionPerformed**.
3. NetBeans jumps to **Source** view and creates an empty method like `mnuExitActionPerformed(...)`, with a `// TODO add your handling code here:` line. Replace that line with the code below.

Do this for all three items. Then go back to **Design** with the **Design** button.

### 8.1 Add one field and the imports

Switch to **Source** view. These go in the white (editable) areas, not the grey ones.

**Imports**, under the `package` line at the top:

```java
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import javax.swing.JOptionPane;
```

**Field**, just inside the class, above the constructor:

```java
private final MovieTickets movieTickets = new MovieTickets();
```

### 8.2 Exit (Q.2.3)

```java
private void mnuExitActionPerformed(java.awt.event.ActionEvent evt) {
    System.exit(0);
}
```

### 8.3 Process (Q.2.4, Q.2.5 and Q.2.7)

This captures the inputs, rejects anything invalid, works out the total with VAT, shows the report and saves it.

```java
private void mnuProcessActionPerformed(java.awt.event.ActionEvent evt) {
    String movieName = cmbMovie.getSelectedItem() == null ? "" : cmbMovie.getSelectedItem().toString();

    int numberOfTickets;
    double ticketPrice;
    try {
        numberOfTickets = Integer.parseInt(txtNumberOfTickets.getText().trim());
    } catch (NumberFormatException e) {
        JOptionPane.showMessageDialog(this, "Please enter a whole number for the number of tickets.",
                "Invalid input", JOptionPane.ERROR_MESSAGE);
        txtNumberOfTickets.requestFocus();
        return;
    }
    try {
        ticketPrice = Double.parseDouble(txtTicketPrice.getText().trim());
    } catch (NumberFormatException e) {
        JOptionPane.showMessageDialog(this, "Please enter a valid number for the ticket price.",
                "Invalid input", JOptionPane.ERROR_MESSAGE);
        txtTicketPrice.requestFocus();
        return;
    }

    // Only valid data is captured
    MovieTicketData data = new MovieTicketData(movieName, numberOfTickets, ticketPrice);
    if (!movieTickets.ValidateData(data)) {
        JOptionPane.showMessageDialog(this, movieTickets.getValidationMessage(data),
                "Invalid input", JOptionPane.ERROR_MESSAGE);
        return;
    }

    double totalTicketPrice = movieTickets.CalculateTotalTicketPrice(numberOfTickets, ticketPrice);

    String report = "MOVIE NAME: " + movieName + "\n"
            + "MOVIE TICKET PRICE: R " + ticketPrice + "\n"
            + "NUMBER OF TICKETS: " + numberOfTickets + "\n"
            + "TOTAL TICKET PRICE: R " + totalTicketPrice;
    txaReport.setText(report);

    saveReport(report);
}
```

The two `try/catch` blocks matter. Without them, typing letters such as `abc` in a text field would crash the program with a `NumberFormatException` instead of showing a friendly message.

### 8.4 Clear (Q.2.6)

```java
private void mnuClearActionPerformed(java.awt.event.ActionEvent evt) {
    cmbMovie.setSelectedIndex(0);
    txtNumberOfTickets.setText("");
    txtTicketPrice.setText("");
    txaReport.setText("");
    txtNumberOfTickets.requestFocus();
}
```

### 8.5 Save the report to report.txt (Q.2.5)

Add this method anywhere in the white area of the class. Putting it just after the three handlers keeps it easy to find.

```java
private void saveReport(String report) {
    String stars = "*".repeat(37);
    try (PrintWriter writer = new PrintWriter(new FileWriter("report.txt"))) {
        writer.println("MOVIE TICKET REPORT");
        writer.println(stars);
        writer.println(report);
        writer.println(stars);
    } catch (IOException e) {
        JOptionPane.showMessageDialog(this, "The report could not be saved: " + e.getMessage(),
                "Save error", JOptionPane.ERROR_MESSAGE);
    }
}
```

The `try (...)` form is called *try-with-resources*. It closes the file automatically, even if something goes wrong.

---

## Step 9: Tell Maven which class to run

You deleted the starter class in Step 1, so point the project at the form:

1. Right-click the project and choose **Properties**.
2. Go to **Run**, click **Browse...** next to **Main Class** and pick `com.mycompany.movieticketsapp.MovieTicketsForm`.
3. Click **OK**.

This updates the `exec.mainClass` line in pom.xml for you.

---

## Step 10: Run and check every requirement

Press **F6**, or right-click the project and choose **Run**.

| Test | What to do | What you should see |
|---|---|---|
| Normal processing | Napoleon, `3` tickets, price `100`, then **Tools > Process** | Report shows `TOTAL TICKET PRICE: R 342.0` (300 + 14% VAT) |
| File saved | Open the **Files** tab (next to **Projects**) and refresh the **MovieTicketsApp** folder | `report.txt`, with the stars layout from the sample |
| Tickets = 0 | Tickets `0`, price `100`, **Process** | Message: number of tickets must be greater than zero |
| Price negative | Tickets `2`, price `-50`, **Process** | Message: ticket price must be greater than zero |
| Not a number | Tickets `abc`, **Process** | Friendly message, no crash |
| Read only | Try typing in the report box | Nothing happens |
| Clear | **Tools > Clear** | Fields and report emptied, combo back to Napoleon |
| Exit | **File > Exit** | Application closes |

`report.txt` is written to the folder the program runs from. When you run it from NetBeans, that's the project folder, the same folder that contains `pom.xml`.

---

## Step 11: Unit tests (Q.2.9)

1. In **Projects**, right-click `MovieTickets.java` and choose **Tools > Create/Update Tests** (**Ctrl+Shift+U**).
2. Keep **JUnit 5** (it may be called **JUnit Jupiter**) selected and click **OK**.
3. NetBeans creates **Test Packages > com.mycompany.movieticketsapp > MovieTicketsTest.java**. This is the "test package within the application" the question asks for.
4. Replace its contents with:

```java
package com.mycompany.movieticketsapp;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;
import org.junit.jupiter.api.Test;

public class MovieTicketsTest {

    private final MovieTickets movieTickets = new MovieTickets();

    @Test
    public void CalculateTotalTicketPrice_CalculatedSuccessfully() {
        // 3 tickets x R100 = R300, plus 14% VAT = R342
        double result = movieTickets.CalculateTotalTicketPrice(3, 100);
        assertEquals(342.0, result, 0.001);
    }

    @Test
    public void ValidateData_InvalidNumberOfTickets_ReturnsFalse() {
        MovieTicketData data = new MovieTicketData("Napoleon", 0, 100);
        assertFalse(movieTickets.ValidateData(data));
    }

    @Test
    public void ValidateData_InvalidTicketPrice_ReturnsFalse() {
        MovieTicketData data = new MovieTicketData("Oppenheimer", 2, -50);
        assertFalse(movieTickets.ValidateData(data));
    }

    @Test
    public void ValidateData_EmptyMovieName_ReturnsFalse() {
        MovieTicketData data = new MovieTicketData("", 2, 100);
        assertFalse(movieTickets.ValidateData(data));
    }

    @Test
    public void ValidateData_ValidData_ReturnsTrue() {
        MovieTicketData data = new MovieTicketData("Damsel", 2, 80);
        assertTrue(movieTickets.ValidateData(data));
    }
}
```

The question only asks for one validation test. The extra three cover every rule, which is worth having and costs nothing.

5. Right-click the project and choose **Test** (**Alt+F6**). The **Test Results** window should show **5 tests passed, 0 failed**.

---

## Step 12: Screenshots worth taking for your submission

- The form in **Design** view with the **Navigator** visible. This shows the GUI was built with drag and drop.
- The running app after **Process**, showing the R 342.0 report.
- A validation error message.
- `report.txt` opened in Notepad.
- The **Test Results** window with all tests passing.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| I can't edit some of the code | Grey code is generated by the GUI Builder. Change the layout in **Design** view instead and the code updates itself. |
| An event method disappeared | Never delete a handler method by hand. In Design view, select the item, then in **Properties > Events** clear the handler name. NetBeans removes the method safely. |
| `package org.junit... does not exist` | The JUnit dependency is missing from pom.xml (Step 2). Add it, then **Clean and Build**. |
| "No main class" when running | Do Step 9 again. |
| The Palette is empty | Make sure the `.java` form is open in **Design** view, not Source view. |
| I can't find `report.txt` | Use the **Files** tab and refresh it. The file is in the project's root folder. |
