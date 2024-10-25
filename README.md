import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.regex.*;

public class Editor implements ActionListener {
    private Frame frame;
    private TextArea textArea;
    private boolean lastChange = false;
    private int lastIndex = -1;
    private String lastsearchtext = "";
    private String currentFilePath = null;
    private boolean isFileOpen = false;

    public Editor() {
        frame = new Frame("Editor");
        frame.setSize(500, 500);
        frame.setBackground(Color.WHITE);
        
        MenuBar menuBar = new MenuBar();
        Menu fileMenu = new Menu("File");
        Menu editMenu = new Menu("Edit");

        MenuItem newItem = new MenuItem("New");
        MenuItem openItem = new MenuItem("Open");
        MenuItem saveItem = new MenuItem("Save");
        MenuItem saveAsItem = new MenuItem("Save As");
        MenuItem closeItem = new MenuItem("Close");
        MenuItem findItem = new MenuItem("Find");
        MenuItem findReplaceItem = new MenuItem("Find & Replace");
    

        newItem.addActionListener(this);
        openItem.addActionListener(this);
        saveItem.addActionListener(this);
        saveAsItem.addActionListener(this);
        closeItem.addActionListener(this);
        findItem.addActionListener(this);
        findReplaceItem.addActionListener(this);

        fileMenu.add(newItem);
        fileMenu.add(openItem);
        fileMenu.add(saveItem);
        fileMenu.add(saveAsItem);
        fileMenu.addSeparator();
        fileMenu.add(closeItem);

        editMenu.add(findItem);
        editMenu.add(findReplaceItem);

        menuBar.add(fileMenu);
        menuBar.add(editMenu);


        frame.setMenuBar(menuBar);

        textArea = new TextArea();
        
        textArea.addTextListener(new TextListener() {
            public void textValueChanged(TextEvent e) {
                if (isFileOpen) {
                    isFileOpen = false;
                } else {
                    lastChange = true;
                }
            }
        });

        frame.add(textArea);

        frame.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                handleClose();
            }
        });


        frame.setVisible(true);
        lastChange = false;
    }

    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        switch (command) {
            case "New":
                handleNew();
                break;
            case "Open":
                handleOpen();
                break;
            case "Save":
                handleSave();
                break;
            case "Save As":
                handleSaveAs();
                break;
            case "Close":
                handleClose();
                break;
            case "Find":
                handleFind();
                break;
            case "Find & Replace":
                handleFindReplace();
                break;
            default:
                System.out.println("Unknown command: " + command);
                break;
        }
    }

    private void handleNew() {
        if (lastChange) {
            final Dialog errorDialog = new Dialog(frame, "Unsaved Changes", true);
            errorDialog.setLayout(new FlowLayout());
            errorDialog.setSize(300, 150);

            Label message = new Label("You have unsaved changes. Do you want to save them?");
            Button saveButton = new Button("Save");
            Button dontSaveButton = new Button("Don't Save");
            Button cancelButton = new Button("Cancel");

            saveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    handleSave();
                    errorDialog.dispose();
                    textArea.setText(""); 
                    lastChange = false; 
                    currentFilePath = null;
                }
            });

            dontSaveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    errorDialog.dispose();
                    textArea.setText(""); 
                    lastChange = false; 
                    currentFilePath = null;
                }
            });

            cancelButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    lastChange=false;
                    errorDialog.dispose(); 
                }
            });

            errorDialog.add(message);
            errorDialog.add(saveButton);
            errorDialog.add(dontSaveButton);
            errorDialog.add(cancelButton);

            errorDialog.addWindowListener(new WindowAdapter() {
                public void windowClosing(WindowEvent e) {
                    lastChange=false;
                    errorDialog.dispose();
                }
            });

            errorDialog.setVisible(true);
    }
        else {
            textArea.setText("");
            lastChange = false; 
            currentFilePath = null;
        }
        System.out.println(lastChange);
    }

    private void handleOpen() {
        if (lastChange) {
            System.out.println(lastChange);
            final Dialog errorDialog = new Dialog(frame, "Unsaved Changes", true);
            errorDialog.setLayout(new FlowLayout());
            errorDialog.setSize(300, 150);

            Label message = new Label("You have unsaved changes. Do you want to save them?");
            Button saveButton = new Button("Save");
            Button dontSaveButton = new Button("Don't Save");
            Button cancelButton = new Button("Cancel");

            saveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    handleSave();
                    lastChange = false;
                    errorDialog.dispose();
                    openDialogBox();
                }
            });

            dontSaveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    errorDialog.dispose();
                    lastChange = false;
                    openDialogBox();
                }
            });

            cancelButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    lastChange=false;
                    errorDialog.dispose(); 
                }
            });

            errorDialog.add(message);
            errorDialog.add(saveButton);
            errorDialog.add(dontSaveButton);
            errorDialog.add(cancelButton);

            errorDialog.addWindowListener(new WindowAdapter() {
                public void windowClosing(WindowEvent e) {
                    lastChange=false;
                    errorDialog.dispose();
                }
            });

            errorDialog.setVisible(true);
        } else {
            openDialogBox();
        }       
    }
    
    private void openDialogBox() {
        System.out.println(lastChange);
        FileDialog fileDialog = new FileDialog(frame, "Open File", FileDialog.LOAD);
        fileDialog.setVisible(true);

        String filePath = fileDialog.getDirectory() + fileDialog.getFile();
        if (filePath != null && !filePath.isEmpty()) {
            File file = new File(filePath);
            try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
                StringBuilder content = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    content.append(line).append("\n");
                }
                isFileOpen = true;
                textArea.setText(content.toString());
                lastChange = false;
                currentFilePath = filePath;
            } catch (IOException e) {
                System.out.println("Error reading file: " + e.getMessage());
            }
        }
        System.out.println(lastChange);
    }

    private void handleSave() {
        if (currentFilePath == null) {
            handleSaveAs();
        } else {
            try (BufferedWriter writer = new BufferedWriter(new FileWriter(currentFilePath))) {
                writer.write(textArea.getText());
                lastChange = false;
            } catch (IOException e) {
                System.out.println("Error writing file: " + e.getMessage());
            }
        }
    }

    private void handleSaveAs() {
        FileDialog fileDialog = new FileDialog(frame, "Save File", FileDialog.SAVE);
        fileDialog.setVisible(true);

        String filePath = fileDialog.getDirectory() + fileDialog.getFile();
        if (filePath != null && !filePath.isEmpty()) {
            currentFilePath = filePath;
            try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath))) {
                writer.write(textArea.getText());
                lastChange = false;
            } catch (IOException e) {
                System.out.println("Error writing file: " + e.getMessage());
            }
        }
    }

    private void handleClose() {
        if (lastChange) {
            final Dialog errorDialog = new Dialog(frame, "Unsaved Changes", true);
            errorDialog.setLayout(new FlowLayout());
            errorDialog.setSize(300, 150);

            Label message = new Label("You have unsaved changes. Do you want to save them before closing?");
            Button saveButton = new Button("Save");
            Button dontSaveButton = new Button("Don't Save");
            Button cancelButton = new Button("Cancel");

            saveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    handleSave();
                    errorDialog.dispose();
                    frame.dispose();
                }
            });

            dontSaveButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    errorDialog.dispose();
                    frame.dispose();
                }
            });

            cancelButton.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent e) {
                    errorDialog.dispose();
                }
            });

            errorDialog.add(message);
            errorDialog.add(saveButton);
            errorDialog.add(dontSaveButton);
            errorDialog.add(cancelButton);

            errorDialog.addWindowListener(new WindowAdapter() {
                public void windowClosing(WindowEvent e) {
                    errorDialog.dispose();
                }
            });

            errorDialog.setVisible(true);
        } else {
            frame.dispose();
        }
    }

    private void handleFind() {
        final Dialog findDialog = new Dialog(frame, "Find", false); // Modeless dialog
        findDialog.setLayout(new FlowLayout());
        findDialog.setSize(300, 200);

        Label findLabel = new Label("Find:");
        TextField findField = new TextField(20);
        Button findNextButton = new Button("Find Next");
        Button closeButton = new Button("Close");

        findDialog.add(findLabel);
        findDialog.add(findField);
        findDialog.add(findNextButton);
        findDialog.add(closeButton);

        findDialog.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                findDialog.dispose();
            }
        });

        findNextButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String searchText = findField.getText();
                if (!lastsearchtext.equals(searchText)) {
                    lastIndex = -1;
                }
                if (!searchText.isEmpty()) {
                    String content = textArea.getText();
                    String lowerContent = content.toLowerCase();
                    String lowerSearchText = searchText.toLowerCase();
                    int startIndex = (lastIndex != -1) ? lastIndex + searchText.length() : 0;
                    int index = lowerContent.indexOf(lowerSearchText, startIndex);
                    if (index == -1) {
                        showErrorDialog("No more occurrences found.");
                    } else {
                        lastIndex = index;
                        lastsearchtext = searchText;
                        textArea.select(index, index + searchText.length());
                        textArea.requestFocus();
                        textArea.repaint();
                    }
                }
            }
        });

        closeButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                findDialog.dispose();
            }
        });

        findDialog.setVisible(true);
    }

    private void handleFindReplace() {
        final Dialog replaceDialog = new Dialog(frame, "Find & Replace", false); // Modeless dialog
        replaceDialog.setLayout(new FlowLayout());
        replaceDialog.setSize(300, 200);

        Label findLabel = new Label("Find:");
        TextField findField = new TextField(20);
        Label replaceLabel = new Label("Replace with:");
        TextField replaceField = new TextField(20);
        Button findNextButton = new Button("Find Next");
        Button replaceButton = new Button("Replace");
        Button replaceAllButton = new Button("Replace All");
        Button cancelButton = new Button("Cancel");

        replaceDialog.add(findLabel);
        replaceDialog.add(findField);
        replaceDialog.add(replaceLabel);
        replaceDialog.add(replaceField);
        replaceDialog.add(findNextButton);
        replaceDialog.add(replaceButton);
        replaceDialog.add(replaceAllButton);
        replaceDialog.add(cancelButton);

        findNextButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String searchText = findField.getText();
                if (!lastsearchtext.equals(searchText)) {
                    lastIndex = -1;
                }
                if (!searchText.isEmpty()) {
                    String content = textArea.getText();
                    String lowerContent = content.toLowerCase();
                    String lowerSearchText = searchText.toLowerCase();
                    int startIndex = (lastIndex != -1) ? lastIndex + searchText.length() : 0;
                    int index = lowerContent.indexOf(lowerSearchText, startIndex);
                    if (index == -1) {
                        showErrorDialog("No more occurrences found.");
                    } else {
                        lastIndex = index;
                        lastsearchtext = searchText;
                        textArea.select(index, index + searchText.length());
                        textArea.requestFocus();
                        textArea.repaint();
                    }
                }
            }
        });

        replaceButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String searchText = findField.getText();
                String replaceText = replaceField.getText();
                if (!searchText.isEmpty()) {
                    String content = textArea.getText();
                    int index = content.toLowerCase().indexOf(searchText.toLowerCase());

                    if (index != -1) {
                        textArea.setText(content.substring(0, index) + replaceText + content.substring(index + searchText.length()));
                        textArea.requestFocus();
                        lastIndex = index;
                    } else {
                        showErrorDialog("Text not found.");
                    }
                }
            }
        });

        replaceAllButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String searchText = findField.getText();
                String replaceText = replaceField.getText();
                if (!searchText.isEmpty()) {
                    String content = textArea.getText();
                    if (content.toLowerCase().contains(searchText.toLowerCase())) {
                        String updatedContent = content.replaceAll("(?i)" + Pattern.quote(searchText), Matcher.quoteReplacement(replaceText)); // Case-insensitive replace
                        textArea.setText(updatedContent);
                        textArea.requestFocus();
                        replaceDialog.dispose();
                    } else {
                        showErrorDialog("Text not found for replacement.");
                    }
                }
            }
        });

        cancelButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                replaceDialog.dispose();
            }
        });

        replaceDialog.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                replaceDialog.dispose();
            }
        });

        replaceDialog.setVisible(true);
    }

    private void showErrorDialog(String message) {
        Dialog errorDialog = new Dialog(frame, "Error", true);
        errorDialog.setLayout(new FlowLayout());
        errorDialog.setSize(300, 150);
        Label errorLabel = new Label(message);
        Button okButton = new Button("OK");
        errorDialog.add(errorLabel);
        errorDialog.add(okButton);

        okButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                errorDialog.dispose();
            }
        });

        errorDialog.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                errorDialog.dispose();
            }
        });

        errorDialog.setVisible(true);
    }

    public static void main(String[] args) {
        new Editor();
    }
}
