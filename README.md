# PLSQL-Test-ABANS

-- Logic 1 --(Data to use : exchange_data.csv and depository_data.csv)

CREATE OR REPLACE PACKAGE BODY PLSQLTEST AS

 CREATE OR REPLACE PROCEDURE ProcParent (F1 in file_type,
                                         F2 in file_type) IS
    
    f1ExchangeData   utl_file.file_type;
    f2RepositoryData utl_file.file_type;
    Var1Exch         varchar2(100);
    Var2Repo         varchar2(100);
    Error1           Exception;
    Error2           Exception;
    
      BEGIN
        f1ExchangeData   := utl_file.fopen('Dir1','exchange_data.csv','r');
        
        LOOP
          utl_file.get_line(f1ExchangeData,Var1Exch);
          BEGIN
           INSERT INTO EXCHANGE_DATA(User_ID, Stock_ID, Stock_Name, Stock_Count)
           VALUES(Var1Exch(1), Var1Exch(2), Var1Exch(3), Var1Exch(4);
           COMMIT;
          EXCEPTION
            WHEN OTHERS THEN
            RAISE Error1;
          END;
          dbms_output.put_line(Var1Exch)
        END LOOP;
      EXCEPTION
        WHEN NO_DATA_FOUND THEN
         utl_file.fclose(f1ExchangeData);
      END;  

  f2RepositoryData := utl_file.fopen('Dir2','depository_data.csv','r');
  
     LOOP
          utl_file.get_line(f2RepositoryData,Var2Repo);
          BEGIN
           INSERT INTO REPOSITORY_DATA(User_ID, Stock_ID, Stock_Name, Stock_Count)
           VALUES(Var2Repo(1), Var2Repo(2), Var2Repo(3), Var2Repo(4);
           COMMIT;
          EXCEPTION
            WHEN OTHERS THEN
            RAISE Error2;
          END;
          dbms_output.put_line(Var2Repo)
        END LOOP;
      EXCEPTION
        WHEN NO_DATA_FOUND THEN
         utl_file.fclose(f2RepositoryData);
      END;  
      
 END ProcParent:
                              




END PLSQLTEST;



--Logic 2
