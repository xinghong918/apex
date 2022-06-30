# Document Flow

## Prerequisite
Supporting OCI services
- Oracle Content and Management Cloud
- Oracle APEX on Autonomous Database

Init Steps
1. Import plug-in [**Oracle APEX Region Plugin - APEX Signature**](https://github.com/Dani3lSun/apex-plugin-apexsignature)
1. Learn APEX Signature [*How to use*](https://github.com/Dani3lSun/apex-plugin-apexsignature#how-to-use) section
1. Customize PL/SQL code for *Save to DB using PL/SQL* and *Get files from default PL/SQL code* purposes
    1. Create table to persist the signature
    
    ```
    create sequence SIGNATURE_seq
        start with 1
        increment by 1
        nomaxvalue
        minvalue 1
        nocycle;
        
    CREATE TABLE docu.signature (
    file_name VARCHAR2(100),
    mime_type VARCHAR2(100),
    date_created VARCHAR2(100),
    img_content BLOB
    );
    
    ```
    1. Customize the PL/SQL snippet for *Save to DB using PL/SQL*
    ```
    DECLARE
    --
    l_collection_name VARCHAR2(100);
    l_clob            CLOB;
    l_blob            BLOB;
    l_filename        VARCHAR2(100);
    l_mime_type       VARCHAR2(100);
    l_token           VARCHAR2(32000);
    --
    BEGIN
    -- get defaults
    l_filename  := 'signature_' ||
                    to_char(SYSDATE,
                            'YYYYMMDDHH24MISS') || '.png';
    l_mime_type := 'image/png';
    -- build CLOB from f01 30k Array
    dbms_lob.createtemporary(l_clob,
                            FALSE,
                            dbms_lob.session);

    FOR i IN 1 .. apex_application.g_f01.count LOOP
        l_token := wwv_flow.g_f01(i);
    
        IF length(l_token) > 0 THEN
        dbms_lob.writeappend(l_clob,
                            length(l_token),
                            l_token);
        END IF;
    END LOOP;
    --
    -- convert base64 CLOB to BLOB (mimetype: image/png)
    l_blob := apex_web_service.clobbase642blob(p_clob => l_clob);
    
    -- insert (only if BLOB not null)
    IF dbms_lob.getlength(lob_loc => l_blob) IS NOT NULL THEN
        insert into SIGNATURE values(
            l_filename,
            l_mime_type,
            SYSDATE,
            l_blob
        );
        
    END IF;
    --
    END;
    ```
    1. Customerize the PL/SQL to list
    ```
    return q'~
    SELECT file_name,
        mime_type,
        date_created,
        img_content
    FROM SIGNATURE
    ;
    ~';
    ```
    


