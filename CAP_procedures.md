1. first create a folder inside db folder called src
                ---> then insede src folder create below files
                        ---> .hdiconfig

                                {
                                "file_suffixes": {
                                    "csv": {
                                    "plugin_name": "com.sap.hana.di.tabledata.source"
                                    },
                                    "hdbcalculationview": {
                                    "plugin_name": "com.sap.hana.di.calculationview"
                                    },
                                    "hdbconstraint": {
                                    "plugin_name": "com.sap.hana.di.constraint"
                                    },
                                    "hdbindex": {
                                    "plugin_name": "com.sap.hana.di.index"
                                    },
                                    "hdbtable": {
                                    "plugin_name": "com.sap.hana.di.table"
                                    },
                                    "hdbtabledata": {
                                    "plugin_name": "com.sap.hana.di.tabledata"
                                    },
                                    "hdbview": {
                                    "plugin_name": "com.sap.hana.di.view"
                                    },
                                    
                                    "hdbprocedure": {
                                    "plugin_name": "com.sap.hana.di.procedure"
                                    }
                                }
                                }
2. press on your keyboard ctrl + shift + p  and search for  artifects and select 'create hana database artifects'
                    ---> choose your path of source
                    ---> select hdbprocedure
                    ---> give a name
            
    it will create your procedure file then write your program (example)

                        PROCEDURE "proooo"(
                            
                            OUT out_tab TABLE (
                            ID    INT,
                            NAME  NVARCHAR(100)   -- set a length suitable for your data
                        )

                        )
                        LANGUAGE SQLSCRIPT
                        SQL SECURITY INVOKER
                        --DEFAULT SCHEMA <default_schema_name>
                        AS
                        BEGIN
                        /*************************************
                            Write your procedure logic
                        *************************************/
                        out_tab = select ID as id, NAME as name from DATAB_STUDENT;
                        END
