package com.zte.test.thread.threadpooltest;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Scanner;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class CallableMatchCounter implements Callable<Integer> {
    private File directory;
    private String keyword;
    private int count;

    public CallableMatchCounter(File directory, String keyword) {
        this.directory = directory;
        this.keyword = keyword;
    }

    public Integer call() throws Exception {
        count = 0;
        try {
            File[] files = directory.listFiles();
            ArrayList<Future<Integer>> results = new ArrayList<Future<Integer>>();
            for(File file : files) {
                if (file.isDirectory()) {
                    CallableMatchCounter counter = new CallableMatchCounter(file, keyword);
                    FutureTask<Integer> task = new FutureTask<Integer>(counter);
                    results.add(task);
                    Thread t = new Thread(task);
                    t.start();
                } else {
                    if(search(file)) {
                        count++;
                    }
                }
            }
            for (Future<Integer> result : results) {
                try {
                    count += result.get();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
            }
        } catch(InterruptedException e){
            e.printStackTrace();
        }
        return count;
    }
    
    public boolean search(File file) {
        try {
            Scanner in = new Scanner(new FileInputStream(file));
            boolean found = false;
            while (!found && in.hasNext()) {
                String line = in.nextLine();
                if (line.contains(keyword)) {
                    System.out.println("Thread: " + Thread.currentThread() + ". " + file.getPath());
                    found = true;
                }
            }
            in.close();
            return found;
        } catch (IOException e) {
            System.out.println("Error!");
            e.printStackTrace();
            return false;
        }
    }
}
