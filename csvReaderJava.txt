import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Scanner;
import java.util.Comparator;
import java.util.Arrays;
// imports are really neat, that's a cool feature of Java
//===================================================
//                    READ ME
//===================================================
//  So this was made in about 8 hours, using lots of online resources and lookups and spending a decent amount of time in the last few days learning the fundamentals of java - that said, first time I've written in .java (though I'm very familiar with JS with a passing familiarty with Python and C).  I'm relatively proud of myself for getting this done in the time I gave myself.  

// The formatting of the code is similar to the namespacing I'm familiar with in JS - I'm sure there's a more object oriented way to structure the entire thing but again, this is very new to me.  Anyway, the code cascades 'upwards' for the most part, with the initializing function at the bottom.  

// I used W3Schools for lots of learning, and especially for the file reading/writing aspects, and then the actual "sorting" aspect of the is an adaptation of some code I found online, I didn't have a good grasp on Combinatorsso I used their code as a framework and built on it.

//  I'm going to be a little lighter on comments than I was in my JS submission, just to keep it readable and because I'm assuming you're more familiar with Java than JS.  Overall, the logic is mostly the same, just with different syntactic approaches (and compensating for some of the crazy things about java, like arrays being a set length, haha)
//
//===================================================
//===================================================

public class Main {    
    public static String[][] getCSVFile(){
        // I tried to figure out a nicer way to do this, but I couldn't find one while keeping in my self imposed time limit,
        // basically I loop through the file twice, once to see how many lines it is, and then again to actually read it and split it. 
            // the reason I needed to find the file line count is because Java doesn't seem to have a nice way to adjust allocated memory for Arrays
            // So, I build an array that will exactly hold the length of the file by counting how long the file is.  Unfortunately it results in some really ugly code where I just call it twice, I would _LOVE_ to know the better way to do this
                // As an aside, I tried using the import ArrayList, but ran into so many problems with sorting I re-wrote my code to just use a nested array.

        int lineCount = 0;
        String[][] failstring = {{"it failed"}};
        // I just have 'failstring' because the try/catch requires a return of the correct type, super sloppy but it works. 

        try{
            // File csvFile = new File ("testCSV.csv");           
            File csvFile = new File ("orderlines.csv");           
            Scanner csvReader = new Scanner(csvFile);
            while(csvReader.hasNextLine()){
                csvReader.nextLine();
                lineCount++;
            }
            csvReader.close();
        } catch (FileNotFoundException e){
            System.out.println("Error on the file catch");
            e.printStackTrace();
            return failstring;
        }

        String[][] dataArray2d = new String [lineCount][3];
        int indexOfData = 0;
        // because the lines correspond to indexes in the array, I can write to the index using the line count;
        // it requires an externally scoped variable because the way I learnted to read/write files uses the try/catch setup

        try{
            // File csvFile = new File ("testCSV.csv");           
            File csvFile = new File ("orderlines.csv");           
            Scanner csvReader = new Scanner(csvFile);

            while(csvReader.hasNextLine()){
                String lineData = csvReader.nextLine();
                String[] lineArray = lineData.split(";");
                dataArray2d[indexOfData] = lineArray;
                indexOfData++;
            }
            csvReader.close();
            return dataArray2d;
        } catch (FileNotFoundException e){
            System.out.println("Error on the file catch");
            e.printStackTrace();
            return failstring;
        }
    }


    public static void sortFunction(int[][] arrToSort, int indexToSort){
        // this is the function that sorts the arrays, I repurposed code for the Comparator that I found online, I'm unfamiliar enough with the syntax and how to use Comparators, the credit to that code is below.
        // Once it uses the magic of the Comparator, it does a similar compare function that is used in JS where it can compare values and shuffle them forward or backwards.  I actually ran into some interesting problems with those code between my test file and the full orderlines, which turned out to be a simple equivilency error - why it failed for file lengths over ~6000 lines is beyond me, I had no issues in my test file but orderlines.csv was not having it...

        // code adapted from https://www.geeksforgeeks.org/sorting-2d-array-according-values-given-column-java/

        Arrays.sort(arrToSort, new Comparator<int[]>(){
            public int compare(final int[] a, final int[]b){
                if (a[indexToSort] > b[indexToSort]){
                    return 1;
                } else if ((a[indexToSort] < b[indexToSort])){
                    return -1;
                } else {
                    return 0;
                }
            }
        });
    }

    public static int[][] condenseFunction(int[][]sortedArr, int arrLength){
        // the logic for this is similar to my logic in my JS solution,
            // basically, if we know the array is ordered in a certain way, we can just 'remember' the last value and look for a change, then push the result to a new array.
            // a big difference is that, because of memory allocation issues, I had to figure out the array length beforehand and pass that in so we end up with a correctly sized array.
        int currentId = sortedArr[0][0];
        int ongoingSum = 0;
        int newLen = 0;
        int [][]condensedArr = new int [arrLength][2];

        for (int i=0; i < arrLength; i++){
            int productID = sortedArr[i][0];
            int productCount = sortedArr[i][1]; 
            if (productID == currentId){
                ongoingSum = ongoingSum + productCount;
            } else {
                condensedArr[newLen][0] = currentId;
                condensedArr[newLen][1] = ongoingSum;
                ongoingSum = productCount;
                currentId = productID;
                newLen++;
                // the newlen increment is what cycles index numbers for the arrays, which is bound on the upper end by the arrlength int parameter
            }
        }
        return condensedArr;
    }

    public static int getNewLength(int[][] condensedArr, int originalLength){
        // this function is to find and remove the PRODUCT_ID = 0, QTY = 0;
        // I feel like this is slightly inelegant but it solves the memory address and array length problem nicely.  
            // the obvious issue is that if we wanted to know that product 0 has qty of 0 it will skip over that edge case.  

        int count = 0;
        // original length - 1 because index starts at 0
        for (int i = (originalLength - 1); i >= 0; i--){
            if ((condensedArr[i][0] != 0) && (condensedArr[i][1] != 0)){
                count++;
            } else {
                break;
                // breaks out when it finds zeros, which we know will be the end because the array it's scanning is sorted.
            }
        }
        return count;
    }

    public static String[][] finalSort(int[][] condensedArr, int smallLen){
        // this function handles building the array that will be piped into the .csv file at the end by "Backconverting" some of the things I had to pull out of the data to properly sort it. 
        // namely, it adds on the "Product_" text, the leading zeros (Which it calculates using the length; technically bound by the 5 digits but that's within the scope of the project), and adds in the semicolons and newlines

        String[][] finalSortedArr = new String[smallLen][2];
        int j = 0;
        for (int i = (condensedArr.length - 1); i >= (condensedArr.length - smallLen); i--){
            // this is counting down through the index based on the length of the array coming in, and it's smallest size, so it can reverse the order of the array while it adds everything in.  

            int numDigits = (int) (Math.log10(condensedArr[i][0])+1);
            // neat trick I read about to get the digits in a computationally more optimal way - my first thought was a string conversion but this is way better. 

            String zeroCount = "";
            for (int z = 0; z < (5 - numDigits); z++){
                zeroCount = zeroCount + "0";
            }

            finalSortedArr[j][0] = "Product_" + zeroCount + Integer.toString(condensedArr[i][0]) + ";";
            finalSortedArr[j][1] = Integer.toString(condensedArr[i][1]) + "\n";
            j++;
        }
        return finalSortedArr;
    }

    public static void printToFile(String[][] arrToPrint){
        // the function that actually writes to a new .csv file; again a lot of this code is heavily adapted from lessons and howtos, mostly on W3schools, as this is pure syntactic stuff that I am very new to. 

        String printMe = "";
        // the string that will be pushed to the file ultimately,

        for (String[] line : arrToPrint){
            for(String entry : line){
                // for each loop that goes through every line, then through every item in that line, then adds it to the printMe string variable.
                printMe = printMe + entry;
            }
        } 

        try{
            // this try is what creates the file
            File finalFile = new File("ABC.csv");
            if(finalFile.createNewFile()){
                finalFile.getName();
                // System.out.println("file created " + finalFile.getName());
            }
        } catch (IOException  e){
            e.printStackTrace();
        }

        try{
            // this try/catch is what writes to it.
            FileWriter finalFileWrite = new FileWriter("ABC.csv");
            finalFileWrite.write(printMe);
            finalFileWrite.close();
        } catch(IOException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String[][] array2D = getCSVFile();
        // call the file reader to access the file in the folder
        int lenOfCSVdata = array2D.length;
        // looks for the length of the first array, which is used for indexing other arrays
        int[][] arrayToSort = new int [lenOfCSVdata][2];
        // creating the array(s) that will be the correct length(size) to hold all the data

        for (String[] item : array2D){
            // goes through the array and removes the Product_ so it can be converted to / treated as an int
            String cleanedUpValue = item[1].replace("Product_", ""); 
            item[1] = cleanedUpValue;
        }  

        for (int i = 0; i < lenOfCSVdata; i++){
            // converts the product id to an int and populates it, and the qty, to an array that can be more easily sorted
            arrayToSort[i][0] = Integer.parseInt(array2D[i][1]);
            arrayToSort[i][1] = Integer.parseInt(array2D[i][2]);
        }
        sortFunction(arrayToSort, 0);
        // sorts the array by product id
        int[][] condensedArr = condenseFunction(arrayToSort, lenOfCSVdata);
        // condenses the qtys of that array, also passes in the total size of the original array because it's necessary for the loop we need to run (which requires iterating indicies, and not a for/each, because java has that memory address thing going on)
        sortFunction(condensedArr, 1);
        // sorts the function by qty, now that it's condensed
        int smallLen = getNewLength(condensedArr, lenOfCSVdata);
        // gets the length of the useful data by trimming off zeroes
            // if there was one part of this code I think would be reasonable to modify based on my knowledge level it's this, basically by checking and ingoring those 0/0 items in the actual condense function.  The 0/0 items are the difference between the sorted and unsorted data, so it's not insignificant.  I might try and solve this this weekend.  
        String[][] finalSortedArr = finalSort(condensedArr, smallLen);
        // This runs the final sort, which is what reformats the information to something that can be printed

        printToFile(finalSortedArr);
        // finally, the array is printed to ABC.csv
    }       
}