package com.example.demo.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;
import java.io.File;

@Service
public class XmlFolderWatcherService {

    private final XmlProcessorService xmlProcessorService;

    // Path where SBI is dropping the files
    private static final String DIRECTORY_PATH = "/path/to/sbi/folder";

    public XmlFolderWatcherService(XmlProcessorService xmlProcessorService) {
        this.xmlProcessorService = xmlProcessorService;
    }

    // Runs every 2 minutes
    @Scheduled(fixedRate = 2 * 60 * 1000)
    public void watchAndProcessFiles() {
        File folder = new File(DIRECTORY_PATH);

        if (!folder.exists() || !folder.isDirectory()) {
            System.err.println("Directory does not exist: " + DIRECTORY_PATH);
            return;
        }

        File[] files = folder.listFiles((dir, name) -> name.toLowerCase().endsWith(".xml"));
        if (files == null || files.length == 0) {
            System.out.println("No XML files to process.");
            return;
        }

        for (File file : files) {
            try {
                xmlProcessorService.processXmlFile(file.getAbsolutePath());
                boolean deleted = file.delete();
                if (deleted) {
                    System.out.println("Processed and deleted: " + file.getName());
                } else {
                    System.err.println("Failed to delete: " + file.getName());
                }
            } catch (Exception e) {
                System.err.println("Error processing file " + file.getName() + ": " + e.getMessage());
            }
        }
    }
}
