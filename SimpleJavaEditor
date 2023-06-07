package simplejavaeditor;

/**
 *
 * @author PureForWhite
 */
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.KeyEvent;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import javax.swing.text.BadLocationException;
import javax.swing.text.Document;
import javax.swing.undo.UndoManager;
import java.io.*;

public class SimpleJavaEditor extends JFrame {

    private JTextArea textArea;
    private boolean isDirty = false;
    private File currentFile;
    private boolean autoSave = false;
    private String searchText;
    private String replaceText;
    private int searchStartIndex = 0;
    private UndoManager undoManager;

    DocumentListener autoSaveListener = new DocumentListener() {
        private void setDirty(DocumentEvent e) {
            isDirty = true;
            if (autoSave && currentFile != null) {
                save();
            }
        }

        @Override
        public void insertUpdate(DocumentEvent e) {
            setDirty(e);
        }

        @Override
        public void removeUpdate(DocumentEvent e) {
            setDirty(e);
        }

        @Override
        public void changedUpdate(DocumentEvent e) {
            setDirty(e);
        }
    };

    public SimpleJavaEditor() {
        super("Simple Java Editor");

        textArea = new JTextArea();
        JScrollPane scrollPane = new JScrollPane(textArea);
        add(scrollPane, BorderLayout.CENTER);

        undoManager = new UndoManager();
        textArea.getDocument().addUndoableEditListener(evt -> undoManager.addEdit(evt.getEdit()));

        JMenuBar menuBar = new JMenuBar();
        JMenu fileMenu = new JMenu("File");
        JMenuItem newItem = new JMenuItem("New");
        JMenuItem openItem = new JMenuItem("Open");
        JMenuItem saveItem = new JMenuItem("Save (Ctrl+S)");
        JMenuItem saveAsItem = new JMenuItem("Save As");
        JMenuItem searchItem = new JMenuItem("Search (Ctrl+F)");
        JMenuItem replaceItem = new JMenuItem("Replace (Ctrl+R)");
        JMenuItem replaceAllItem = new JMenuItem("Replace All");
        JMenuItem undoItem = new JMenuItem("Undo (Ctrl+Z)");
        JMenuItem redoItem = new JMenuItem("Redo (Ctrl+Y)");
        JCheckBoxMenuItem autoSaveItem = new JCheckBoxMenuItem("Toggle Auto-Save");

        saveItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, ActionEvent.CTRL_MASK));
        searchItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_F, ActionEvent.CTRL_MASK));
        replaceItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_R, ActionEvent.CTRL_MASK));
        undoItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_Z, ActionEvent.CTRL_MASK));
        redoItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_Y, ActionEvent.CTRL_MASK));

        newItem.addActionListener(e -> {
            if (isDirty) {
                saveAs();
            }
            textArea.setText("");
            currentFile = null;
            isDirty = false;
        });

        openItem.addActionListener(e -> openFile());

        saveItem.addActionListener(e -> {
            if (currentFile == null) {
                saveAs();
            } else {
                save();
            }
        });

        saveAsItem.addActionListener(e -> saveAs());

        searchItem.addActionListener(e -> {
            searchText = JOptionPane.showInputDialog("Enter text to search");
            if (searchText != null && !searchText.isEmpty()) {
                int startIndex = textArea.getText().indexOf(searchText);
                if (startIndex >= 0) {
                    textArea.select(startIndex, startIndex + searchText.length());
                    searchStartIndex = startIndex + searchText.length();
                } else {
                    JOptionPane.showMessageDialog(null, "Text not found");
                }
            }
        });

        replaceItem.addActionListener(e -> {
            searchText = JOptionPane.showInputDialog("Enter text to search");
            replaceText = JOptionPane.showInputDialog("Enter text to replace");
            searchStartIndex = 0;
            if (searchText != null && replaceText != null && !searchText.isEmpty()) {
                replaceNextOccurrence();
            }
        });

        replaceAllItem.addActionListener(e -> {
            searchText = JOptionPane.showInputDialog("Enter text to search");
            replaceText = JOptionPane.showInputDialog("Enter text to replace");
            if (searchText != null && replaceText != null && !searchText.isEmpty()) {
                String currentText = textArea.getText();
                if (currentText.contains(searchText)) {
                    textArea.setText(currentText.replace(searchText, replaceText));
                    isDirty = true;
                } else {
                    JOptionPane.showMessageDialog(null, "Text not found");
                }
            }
        });

        undoItem.addActionListener(e -> {
            if (undoManager.canUndo()) {
                undoManager.undo();
            } else {
                JOptionPane.showMessageDialog(null, "Nothing to undo");
            }
        });

        redoItem.addActionListener(e -> {
            if (undoManager.canRedo()) {
                undoManager.redo();
            } else {
                JOptionPane.showMessageDialog(null, "Nothing to redo");
            }
        });

        autoSaveItem.addActionListener(e -> {
            autoSave = !autoSave; // Toggle the autoSave
            autoSaveItem.setText(autoSave ? "Auto-Save On" : "Toggle Auto-Save"); // Change text
            if (autoSave) {
                textArea.getDocument().addDocumentListener(autoSaveListener);
            } else {
                textArea.getDocument().removeDocumentListener(autoSaveListener);
            }
        });

        fileMenu.add(newItem);
        fileMenu.add(openItem);
        fileMenu.add(saveItem);
        fileMenu.add(saveAsItem);
        fileMenu.add(searchItem);
        fileMenu.add(replaceItem);
        fileMenu.add(replaceAllItem);
        fileMenu.add(undoItem);
        fileMenu.add(redoItem);
        fileMenu.add(autoSaveItem);
        menuBar.add(fileMenu);
        setJMenuBar(menuBar);

        setSize(800, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setVisible(true);
    }

    private void openFile() {
        JFileChooser fileChooser = new JFileChooser();
        int option = fileChooser.showOpenDialog(SimpleJavaEditor.this);
        if (option == JFileChooser.APPROVE_OPTION) {
            currentFile = fileChooser.getSelectedFile();
            try {
                textArea.read(new FileReader(currentFile), null);
                isDirty = false;
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    private void replaceNextOccurrence() {
        Document doc = textArea.getDocument();
        try {
            String currentText = doc.getText(0, doc.getLength());
            int startIndex = currentText.indexOf(searchText, searchStartIndex);
            if (startIndex >= 0) {
                int confirm = JOptionPane.showConfirmDialog(null, "Found \"" + searchText + "\". Replace this occurrence?", "Replace", JOptionPane.YES_NO_OPTION);
                if (confirm == JOptionPane.YES_OPTION) {
                    doc.remove(startIndex, searchText.length());
                    doc.insertString(startIndex, replaceText, null);
                    isDirty = true;
                }
                searchStartIndex = startIndex + replaceText.length();
            } else {
                JOptionPane.showMessageDialog(null, "No more matches found");
            }
        } catch (BadLocationException e) {
            e.printStackTrace();
        }
    }

    private void saveAs() {
        JFileChooser fileChooser = new JFileChooser();
        int option = fileChooser.showSaveDialog(SimpleJavaEditor.this);
        if (option == JFileChooser.APPROVE_OPTION) {
            currentFile = fileChooser.getSelectedFile();
            save();
        }
    }

    private void save() {
        if (currentFile != null) {
            try {
                BufferedWriter writer = new BufferedWriter(new FileWriter(currentFile));
                writer.write(textArea.getText());
                writer.close();
                isDirty = false;
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        new SimpleJavaEditor();
    }
}

