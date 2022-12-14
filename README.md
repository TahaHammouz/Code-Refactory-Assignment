
# Code Refactory Assigment

Deleted unused import statements and optimized imports in this section
```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.InputEvent;
import java.awt.event.KeyEvent;
import java.io.*;
```


To further explain their usage without comments, I used the variables previously defined in "APP_NAME" and "MID_CONTAINER". 

```java
    public static  void main(String[] args) {
        new Editor();
    }
    public static final String APP_NAME = "Editor";
    public static final String MID_CONTAINER = "Center";
    public JEditorPane textPanel;
    final private JMenuBar menu;
    public boolean isChanged = false;
    private File file;

```

Rename the previous wrong-naming variables to variables, also rename the BuildMenu,BuildEditMenu and BuildFileMenu to the right way of naming 
```java
    public Editor() {
        super(APP_NAME);
        textPanel = new JEditorPane();
        add(new JScrollPane(textPanel), MID_CONTAINER);
        textPanel.getDocument().addDocumentListener(this);

        menu = new JMenuBar();
        setJMenuBar(menu);
        BuildMenu();
        setVisible(true);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
        private void BuildMenu() {
            BuildFileMenu();
            BuildEditMenu();
        }

```
Changed the method names from lowercase to capital letters to signify that they are methods. As well as fixing variable names such as ( "sall" -> "selectAll"), ( "saveas" -> "saveAs").

```java
    private void BuildEditMenu() {
        JMenu edit = new JMenu("Edit");
        menu.add(edit);
        edit.setMnemonic('E');
        JMenuItem cut = new JMenuItem("Cut");
        cut.addActionListener(this);
        cut.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_X, InputEvent.CTRL_DOWN_MASK));
        cut.setMnemonic('T');
        edit.add(cut);
        JMenuItem copy = new JMenuItem("Copy");
        copy.addActionListener(this);
        copy.setMnemonic('C');
        copy.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_C, InputEvent.CTRL_DOWN_MASK));
        edit.add(copy);
        JMenuItem paste = new JMenuItem("Paste");
        paste.setMnemonic('P');
        paste.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_V, InputEvent.CTRL_DOWN_MASK));
        edit.add(paste);
        paste.addActionListener(this);
        JMenuItem find = new JMenuItem("Find");
        find.setMnemonic('F');
        find.addActionListener(this);
        edit.add(find);
        find.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_F, InputEvent.CTRL_DOWN_MASK));
        JMenuItem selectAll = new JMenuItem("Select All");
        selectAll.setMnemonic('A');
        selectAll.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_A, InputEvent.CTRL_DOWN_MASK));
        selectAll.addActionListener(this);
        edit.add(selectAll);
    }

```

The switch statement was used instead of if and else if. Furthermore, I ditched the magic numbers for "result" and "answer" and used constants from the "JOptionPane" library instead (I used comments to understand their meaning; each number has been replaced with its corresponding constant). I also removed the redundant and unused if statements.

```java

public void actionPerformed(ActionEvent e) {
        String action = e.getActionCommand();
        switch (action) {
            case "Quit":
                System.exit(0);
            case "Open":
                LoadFile();
                break;
            case "Save":
                int answer = JOptionPane.YES_OPTION;
                if (isChanged) {
                    answer = JOptionPane.showConfirmDialog(null, "The file has been changed. You want to save it?", "Save file",
                            JOptionPane.YES_NO_OPTION, JOptionPane.WARNING_MESSAGE);
                }
                if (answer == JOptionPane.YES_OPTION) {
                    if (file == null) {
                        SaveAs("Save");
                    } else {
                        String text = textPanel.getText();
                        System.out.println(text);
                        try (PrintWriter writer = new PrintWriter(file)) {
                            if (!file.canWrite())
                                throw new Exception("Cannot write file!");
                            writer.write(text);
                            isChanged = false;
                        } catch (Exception ex) {
                            ex.printStackTrace();
                        }
                    }
                }
                break;
            case "New":
                if (isChanged) {
                    answer = JOptionPane.showConfirmDialog(null, "The file has been changed. You want to save it?", "Save file",
                            JOptionPane.YES_NO_OPTION, JOptionPane.WARNING_MESSAGE);
                    if (answer == JOptionPane.NO_OPTION)
                        return;

                    if (file == null) {
                        SaveAs("Save");
                        return;
                    }
                    String text = textPanel.getText();
                    System.out.println(text);
                    try (PrintWriter writer = new PrintWriter(file)) {
                        if (!file.canWrite())
                            throw new Exception("Cannot write file!");
                        writer.write(text);
                        isChanged = false;
                    } catch (Exception ex) {
                        ex.printStackTrace();
                    }
                }
                file = null;
                textPanel.setText("");
                isChanged = false;
                setTitle("Editor");
                break;
            case "Save as...":
                SaveAs("Save as...");
                break;
            case "Select All":
                textPanel.selectAll();
                break;
            case "Copy":
                textPanel.copy();
                break;
            case "Cut":
                textPanel.cut();
                break;
            case "Paste":
                textPanel.paste();
                break;
            case "Find":
                FindDialog find = new FindDialog(this, true);
                find.showDialog();
                break;
        }
    }
```

In the previous method, the magic numbers were replaced with numerical values using the same library. I also put the file reading code in a method called "ReadFile" and called it in place of the file reading code. As part of this method, redundant if statements were also removed.
```java
    private void LoadFile() {
        JFileChooser dialog = new JFileChooser(System.getProperty("user.home"));
        dialog.setMultiSelectionEnabled(false);
        try {
            int result = dialog.showOpenDialog(this);
            if (result == JOptionPane.YES_NO_CANCEL_OPTION)
                return;
            if (result == JOptionPane.OK_OPTION) {
                if (isChanged){
                    int answer = JOptionPane.showConfirmDialog(null, "The file has been changed. You want to save it?", "Save file",
                            JOptionPane.YES_NO_OPTION, JOptionPane.WARNING_MESSAGE);
                    if (answer == JOptionPane.NO_OPTION)
                        return;
                    if (file == null) {
                        SaveAs("Save");
                        return;
                    }
                    String text = textPanel.getText();
                    System.out.println(text);
                    try (PrintWriter writer = new PrintWriter(file)){
                        if (!file.canWrite())
                            throw new Exception("Cannot write file!");
                        writer.write(text);
                        isChanged = false;
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
                file = dialog.getSelectedFile();
                //Read file
                StringBuilder rs = new StringBuilder();
                try (	FileReader fr = new FileReader(file);
                         BufferedReader reader = new BufferedReader(fr);) {
                    String line;
                    while ((line = reader.readLine()) != null) {
                        rs.append(line + "\n");
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                    JOptionPane.showMessageDialog(null, "Cannot read file !", "Error !", 0);//0 means show Error Dialog
                }

                textPanel.setText(rs.toString());
                isChanged = false;
                setTitle("Editor - " + file.getName());
            }
        } catch (Exception e) {
            e.printStackTrace();
            //0 means show Error Dialog
            JOptionPane.showMessageDialog(null, e, "Error", 0);
        }
    }

```
Lastly, I renamed the function name to capitalize the first letter of the word and modified the rest 
```java
    private void SaveAs(String dialogTitle) {
        JFileChooser dialog = new JFileChooser(System.getProperty("user.home"));
        dialog.setDialogTitle(dialogTitle);
        int result = dialog.showSaveDialog(this);
        if (result != 0)//0 value if approve (yes, ok) is chosen.
            return;
        file = dialog.getSelectedFile();
        try (PrintWriter writer = new PrintWriter(file);){
            writer.write(textPanel.getText());
            isChanged = false;
            setTitle("Save as Text Editor - " + file.getName());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void insertUpdate(DocumentEvent e) {
        isChanged = true;
    }

    @Override
    public void removeUpdate(DocumentEvent e) {
        isChanged = true;
    }

    @Override
    public void changedUpdate(DocumentEvent e) {
        isChanged = true;
    }

```