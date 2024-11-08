# PLSQL-Test-ABANS

-- Logic 1 --(Data to use : exchange_data.csv and depository_data.csv)

CREATE OR REPLACE PACKAGE BODY PLSQLTEST AS

 CREATE OR REPLACE PROCEDURE ProcParent (F1 in file_type,
                                         F2 in file_type) IS
    
    f1ExchangeData       utl_file.file_type;
    f2RepositoryData     utl_file.file_type;
    Var1Exch_Rec         EXCHANGE_DATA%rowtype;    --Declaring record type to read it from csv file through utl file directory and to insert into table
    Var2Repo_Rec         REPOSITORY_DATA%rowtype;  --Declaring record type to read it from csv file through utl file directory and to insert into table
    Error1               Exception;
    Error2               Exception;
    
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
         utl_file.fclose(f1ExchangeData);
      END;  
--Loading Data from CSV file to REPOSITORY_DATA Table

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
         utl_file.fclose(f2RepositoryData);
      END;  
      
 END ProcParent:
                              




END PLSQLTEST;



--Logic 2
