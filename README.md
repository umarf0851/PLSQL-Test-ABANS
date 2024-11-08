# PLSQL-Test-ABANS

-- Logic 1 --(Data to use : exchange_data.csv and depository_data.csv)

CREATE OR REPLACE PACKAGE BODY PLSQLTEST AS

 CREATE OR REPLACE PROCEDURE ProcParent (F1 in file_type,
                                         F2 in file_type) IS
    f1ExchangeData   utl_file.file_type;
    f2RepositoryData utl_file.file_type;
    Var1Exch         varchar2(100);
    Var2Repo         varchar2(100);
    
      BEGIN
        f1ExchangeData   := utl_file.fopen('Dir1','exchange_data.csv','r');
        f2RepositoryData := utl_file.fopen('Dir2','depository_data.csv','r');
      EXCEPTION

 END ProcParent:
                              




END PLSQLTEST;



--Logic 2
