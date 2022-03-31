# send-post-requst-to-soap-and-rest-api

///Send Post Request in Rest Api

/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
 /*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package testing;

import org.json.*;
import org.json.JSONObject;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.logging.Level;
import org.apache.http.HttpEntity;
import org.apache.http.client.ResponseHandler;
import org.apache.http.client.methods.HttpUriRequest;
import org.apache.http.client.methods.RequestBuilder;
import org.apache.http.entity.mime.HttpMultipartMode;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
/**
 *
 * @author soeunl
 */
public class TestMain {
    
    public static void main(String[] args) throws IOException{
//        Logger.getRootLogger().setLevel(org.apache.log4j.Level.FATAL);
        CloseableHttpClient httpclient = HttpClients.createDefault();
        
        File directory = new File("D:\\Soeun\\process_scanIdCard\\220310_100_id_card_for_test\\");  
        FileWriter myWriter = new FileWriter("C:\\Users\\soeunl\\Desktop\\soeun\\result.txt");
        File fileList[] = directory.listFiles();
        int count = 0;
        
        
        for (File file: fileList) {
          IdCardReader idcard = new IdCardReader();
            try{
             
                //Creating the MultipartEntityBuilder
                MultipartEntityBuilder entitybuilder = MultipartEntityBuilder.create();
                //Setting the mode
                entitybuilder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
                //Setting the mode

                //Adding text to body
                entitybuilder.addTextBody("request_id", "TEST");
                entitybuilder.addBinaryBody("image", file);
                //Building a single entity using the parts
                HttpEntity mutiPartHttpEntity = entitybuilder.build();
                
                //Building the RequestBuilder request object
                RequestBuilder reqbuilder = RequestBuilder.post("http://68.183.184.163:64647/cambodia/ocr");
                reqbuilder.setHeader("Accept", "application/json");
        //            reqbuilder.setHeader("Content-Type", "multipart/form-data;");
                reqbuilder.setHeader( "self_token", "5}uNa/(V@SM+$2<3");
                
                //Set the entity object to the RequestBuilder
                reqbuilder.setEntity(mutiPartHttpEntity);

                //Building the request
                HttpUriRequest multipartRequest = reqbuilder.build();
                ResponseHandler<String> responseHandler = new MyResponseHandler();

                String result = httpclient.execute(multipartRequest, responseHandler);
                
                JSONObject jObject = new JSONObject(result);
                JSONObject getObject = jObject.getJSONObject("ocr_data");
                idcard.setIdNumber(getObject.getString("id"));
                idcard.setName(getObject.getString("name"));
                idcard.setEnSex(getObject.getString("sex"));
                idcard.setEnDob(getObject.getString("birthday_kh"));
                idcard.setAddress(getObject.getString("address"));
                idcard.setImage(file.getName());
                
                 if(getObject.getString("id") == null){
                      fileFailWriter.write(file.getName() + "\n" );
                      if(!fileFailDirectory.exists()){
                          fileFailDirectory.mkdir();
                      }
                      FileUtils.copyFileToDirectory(file, fileFailDirectory);
                  }else{
                      myWriter.write(
                          "fileName:" + file.getName() + ","
                          + "Id:" + getObject.getString("id") + ","
                          + "Name:" + getObject.getString("name") + "," 
                          + "Sex:" + getObject.getString("sex") + "," 
                          + "BirthDay:" +getObject.getString("birthday_kh") + "," 
                          + "Address " + getObject.getString("address"));
                      myWriter.write("\n");
///SAVE TO DATABASE
                      Long success = IdCardDBCommon.addIdCardInfo(idcard, logger);
                      System.out.println("Save Database Result:" + success);

                      lstIdcard.add(idcard);
                  }
                
            }catch(Exception ex){
                System.out.println(ex.getMessage());
            }
            if (count == 5) {
                break;
            }
            count ++;
            System.out.println("file count:" + count);
        }
        myWriter.close();
        System.out.println("ok");
    }
}

//IN IdCardDBCommon class

/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package com.vtc.process.database;

import com.vtc.process.Objects.IdCardReader;
import com.vtc.process.common.Constant;
import static com.vtc.process.database.DbSession.getSequence;
import org.apache.log4j.Logger;
import org.hibernate.Session;
import org.hibernate.Query;

/**
 *
 * @author soeunl
 */
public class IdCardDBCommon{

    public static Long addIdCardInfo(IdCardReader obj, Logger logger) throws Exception{
        Session sessionIdCard = null;
        StringBuilder sqlStr = new StringBuilder();
        sqlStr.append(" insert into id_card_reader ");
        sqlStr.append(" (id, image, create_date, id_number, name, en_sex, en_dob, check_result) ");
        sqlStr.append(" VALUES(?, ?, sysdate ,?, ?, ?, ?, ?)");
        try{
            sessionIdCard = DbSession.getSession(Constant.DB_SCHEMA_CM_PRE);
            sessionIdCard.beginTransaction();
            Long id = getSequence(sessionIdCard,"ID_CARD_READER_SEQ");
            Query query = sessionIdCard.createSQLQuery(sqlStr.toString());
            
            query.setParameter(0, id);
            query.setParameter(1, obj.getImage());
            query.setParameter(2, obj.getIdNumber());
            query.setParameter(3, obj.getName());
            query.setParameter(4, obj.getEnSex());
            query.setParameter(5, obj.getEnDob());
            query.setParameter(6, obj.getCheckResult()); 
            query.executeUpdate();
            sessionIdCard.getTransaction().commit();      
            
            return 1L;
        }catch(Exception ex){
            logger.error("Error addIdCardInfo", ex);
        }finally{
            DbSession.closeObject(sessionIdCard);
        }
        return null;
    }
}

