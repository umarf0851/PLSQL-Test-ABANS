# PLSQL-Test-ABANS

--- Logic 1 --(Data to use : exchange_data.csv and depository_data.csv)  
----Instructions for DBA to create Directories and To Give read,write access to the Session UserName.------------------------
-------------So that these Directories can be accessed by Procedures for further proceedings -----------------------------
-------------------------------------------------------------------------------------------- 
Create DIRECTORY Dir1 AS 'Path of the exchange_data.csv';
Create DIRECTORY Dir2 AS 'Path of the repository_data.csv';

GRANT READ,WRITE ON DIRECTORY Dir1 TO USER;
GRANT READ,WRITE ON DIRECTORY Dir2 TO USER;

-------------------THESE Dir1 and Dir2 is used during loading of files----------------------------------
-------------------------------------------End of Instructions-----------------------


CREATE OR REPLACE PACKAGE BODY PLSQLTEST AS

-- This Procedure Functions as Loading the Data into Tables from CSV files and Validates data consistency and Exception Handling in terms of File Handling............
 
 CREATE OR REPLACE PROCEDURE ProcParentLoad (RetCode OUT VARCHAR2,
                                         RetSTR  OUT VARCHAR2) IS
    
    f1ExchangeData       utl_file.file_type;
    f2RepositoryData     utl_file.file_type;
    Var1Exch_Rec         EXCHANGE_DATA%rowtype;    --Declaring record type to read it from csv file through utl file directory and to insert into table
    Var2Repo_Rec         REPOSITORY_DATA%rowtype;  --Declaring record type to read it from csv file through utl file directory and to insert into table
    Error1               Exception;
    Error2               Exception;

    BEGIN
      BEGIN
      --Loading Data from CSV file to EXCHANGE_DATA Table
      
        f1ExchangeData   := utl_file.fopen('Dir1','exchange_data.csv','r');
        LOOP
          utl_file.get_line(f1ExchangeData,Var1Exch_Rec);          --The rows get stored into record and insertion into table occurs below.
          BEGIN
           INSERT INTO EXCHANGE_DATA(User_ID, Stock_ID, Stock_Name, Stock_Count)
           VALUES(Var1Exch_Rec.User_ID, Var1Exch_Rec.Stock_ID, Var1Exch_Rec.Stock_Name, Var1Exch_Rec.Stock_Count;
           COMMIT;
          EXCEPTION
            WHEN OTHERS THEN
            RAISE Error1;
          END;
          dbms_output.put_line(Var1Exch_Rec)
        END LOOP;
      EXCEPTION
        WHEN NO_DATA_FOUND THEN
         utl_file.fclose(f1ExchangeData);     --Error Handling for Incomplete Rows in the file
      END;  
      
--Loading Data from CSV file to REPOSITORY_DATA Table

 BEGIN 
  f2RepositoryData := utl_file.fopen('Dir2','depository_data.csv','r');
     LOOP
          utl_file.get_line(f2RepositoryData,Var2Repo_Rec);                          --The rows get stored into record and insertion into table occurs below.
          BEGIN
           INSERT INTO REPOSITORY_DATA(User_ID, Stock_ID, Stock_Name, Stock_Count)
           VALUES(Var2Repo_Rec.User_ID, Var2Repo_Rec.Stock_ID, Var2Repo_Rec.Stock_Name, Var2Repo_Rec.Stock_Count;
           COMMIT;
          EXCEPTION
            WHEN OTHERS THEN
            RAISE Error2;
          END;
          dbms_output.put_line(Var2Repo_Rec)
        END LOOP;
      EXCEPTION
        WHEN NO_DATA_FOUND THEN
         utl_file.fclose(f2RepositoryData);     --Error Handling for Incomplete Rows in the file
      END;

      EXCEPTION
     WHEN OTHERS THEN
      RetCode := "E";
      RetSTR  := "ERROR" | SQLEERM;

      --Calling the Below procedure to Execute the whole scenario in single step
     Execute ProcDataProcess(RetCode, 
                              RetSTR);
 END ProcParentLoad;

                              
-- This Procedure functions as Data Processing and Identifying Mismatch, Test Case Scenario's and Log the outcomes into MISMATCH_TABLE with Remarks................
 CREATE OR REPLACE PROCEDURE ProcDataProcess(RetCode OUT VARCHAR2,
                                             RetSTR  OUT VARCHAR2) IS
    COUNT_Exch   NUMBER;
    COUNT_Repo   NUMBER;
    Stock_ID_COUNT_Exch NUMBER;
    Stock_ID_COUNT_Repo NUMBER;
    CURSOR C1 is SELECT * from EXCHANGE_DATA;
    CURSOR C2 is SELECT * from REPOSITORY_DATA;

   BEGIN
   
     FOR i in C1  LOOP
      SELECT C1.Stock_Count, C2.Stock_Count
      INTO   COUNT_Exch, COUNT_Repo
      WHERE C1.User_ID = C2.User_ID AND
            C1.Stock_ID = C2.Stock_ID AND
            C1.Stock_Name = C2.Stock_Name;

--- TEST CASE 1 : Mismatched Stock Count will be inserted into MISMATCH_TABLE (This table will show the difference between Stock Count)

              IF COUNT_Exch <> COUNT_Repo THEN
                  BEGIN
                    INSERT INTO MISMATCH_TABLE (User_ID, Stock_ID,Stock_Name,Exch_Stock_Count, Repo_Stock_Count, REMARKS)
                    VALUES (i.User_ID, i.Stock_ID, i.Stock_Name, COUNT_Exch, COUNT_Repo, "Quantity of Stock Counts mismatch between two Sources" );
                    COMMIT;
                  EXCEPTION
                  WHEN OTHERS THEN
                   RetCode := "E";
                   RetSTR := "ERROR" | i.User_ID | i.Stock_ID | i.Stock_Name ;    
                  END;
              END IF;
      
      END LOOP;
      
-- TEST CASE 2 : Mismatch of Stock ID for USER IDs

   FOR j in C1 LOOP 
     SELECT COUNT(C1.Stock_ID), COUNT(C2.Stock_ID)
      INTO  Stock_ID_COUNT_Exch, Stock_ID_COUNT_Repo
      WHERE C1.User_ID = C2.User_ID;

            IF Stock_ID_COUNT_Exch <> Stock_ID_COUNT_Repo THEN
                
                BEGIN
                    INSERT INTO MISMATCH_TABLE (User_ID, Stock_ID,Stock_Name,Exch_Stock_Count, Repo_Stock_Count, REMARKS)
                    VALUES (i.User_ID, "Mismatch", i.Stock_Name, Stock_ID_COUNT_Exch, Stock_ID_COUNT_Repo, "Quantity of Stock_IDS mismatch between two Sources" );
                    COMMIT;
                  EXCEPTION
                  WHEN OTHERS THEN
                   RetCode := "E";
                   RetSTR := "ERROR" | i.User_ID  ;    
                  END;

            END IF;

   END LOOP;

   EXCEPTION
      WHEN OTHERS THEN
       RetCode := "E";
       RetSTR := "ERROR" | SQLERRM ;
       
 END ProcDataProcess;
 
END PLSQLTEST;



-----------------Logic 2--------------- In this Logic SQL Loader can be used to Load Data into the Tables through Command Promt.
        --------------------                             But in Logic 1 Operations are done using Utl Directiories.
 -- Logic 3 can be used through Oracle Data integrator (ODI) tools, But it depends on Oracle versions to which logic to use versions like 19c is best suitable for Logic 2 ------------------------------------
 ------------------ 
                                               
