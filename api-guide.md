# Modeling through the API

An Application Programming Interface (API) is a standard environment, including tools, protocols, and other routines, in which programs can be written.

Software vendors often provide an API to their customers so they can integrate functionality into their own custom applications. Often, these customers have their own in-house software development team which is necessary since the APIs are intended for use by programmers and not for end-users.

For the Modeling Node, the API is in the form of a dll file called lv_proc.dll 

> [!WARNING]
>
> Follow the instructions in this chapter only if you are very experienced in the use of APIs. If you are not sure, do not attempt to work with this API. 
>

## API details
This dll exposes one function that has the following definition:

​	int nDllRunCmd( char *pszCmd, int nInSize, void *pvData, int *pnOutSize, char pcErrorBuffer[] );

where 

- **pszCmd** is a char pointer to the command you wish to implement.
- **nInSize** is the size of pvData buffer (in bytes). 
- **pvData** is a buffer passed in by the user. The API will write data to this buffer (for example, information requested through a Database AttributeGet command). 
- **pnOutSize** is the size (in bytes) of the data written to the pvData buffer by the API. 
- **pcErrorBuffer** is a char buffer of at least 512 bytes that will receive error messages returned from the API. If you do not wish to retrieve error messages, pass NULL.

Currently, the Database AttributeGet command is the only the Modeling Node command that returns information back to the user. Therefore, the variables nInSize, pvData, and pnOutSize are meaningful only when processing the Database AttributeGet command. These variable values will be ignored when processing all other commands.

When implementing the Database AttributeGet command, if the size of buffer pvData is insufficient to hold all of the data being returned by the dll, an error will be generated with an appropriate message indicating the insufficient buffer size.

The API is contained in the file lv_proc.dll. The following supporting files are also required in order for the API to function properly:

- lv_auth.dll 
- lv_xcry.dll 
- lvstr.msg

Place the dll files where your application can access them (in accordance with how Windows searches for dll files).

The message file (lvstr.msg) should be placed in one of the following locations:

- in the corresponding language folder of the Current Working folder (for example, C:\My_Application\en)
- in the actual current working folder in which your application resides
- in a folder specified by the PATH Environment variable 

The klxproc application retains state information in the form of static variables (for example, connection information and symbol lists) until the application is terminated. Similarly, the dll retains state information from the time the dll is loaded (through a call to LoadLibrary) until it is freed (FreeLibrary). When the dll is freed from the application, any connected user will be disconnected. 

> [!NOTE]
>
> This API was designed solely for single-threaded applications. A thread-safe version of the API is planned for a future release. 
>

## Sample code using C 
The following sample C code illustrates how to implement the dll. 

    #include <stdlib.h>
    #include <stdio.h>
    #include <windows.h>
    
    int main ( int argc, char *argv[] )
    {
        int nRet = 0, nInSize = 0, nOutSize = 0;
        char pcErrorBuf[512] = {0};
        char pcData[ 512 ] = {0};
        HINSTANCE	hLib = NULL;
        FARPROC	fpRunCmd = NULL;
    
        hLib = LoadLibrary ( "lv_proc.dll" );
        
        if ( hLib != NULL )
        {
            fpRunCmd = GetProcAddress ( hLib, "nDllRunKlxCmd");
            if ( fpRunCmd != NULL )
            {
                nRet = fpRunCmd( "connect demo/demo dt222725 10051 demo", 0, (void *)0, (int *)0, pcErrorBuf );
                if ( nRet == 0 )
                {
                    nRet = fpRunCmd( "DATABASE ATTRIBUTEGET user kugpapplst testuser1", 512, (void*)pcData, &nOutSize, pcErrorBuf );
                    if ( (nRet == 0) && nOutSize )
                    {
                        printf( "Attribute value retrieved: %s\n", pcData );
                    }
                    fpRunCmd( "disconnect", pcErrorBuf );
                }
            }
            else
            {
                nRet = 1;
            }
            FreeLibrary ( hLib );
        }
    
        if ( nRet == 0 )
        {
            printf( "succeeded\n" );
        }
        else
        {
            printf( "failed, %d\n", nRet );
            if ( pcErrorBuf[0] )
            {
                printf( pcErrorBuf );
            }
        }
        return nRet;
    }

## Sample code using Visual Basic

The following code segments show how to link the dll in a Visual Basic application:

    Private Declare Function nDllRunKlxCmd Lib "lv_proc.dll" (ByVal pszCmd As String, ByVal nSizeIn As Long, _
        pvData As Any, ByRef nSizeOut As Long, ByVal pcErr As String) As Long
    
    ...
    
    Private Sub buttonProcess_Click()
        Dim nRetOut As Long, nSizeOut As Long
        Dim nDataIn As Long
        Dim dfDataIn As Double
        Dim pcDataIn As String
        Dim pcErrOut As String
        Dim pcCmd As String
        pcErrOut = Space(512)
        pcDataIn = Space(1024)
        nSizeOut = 0
        nRetOut = 0
    
        ...
        pcCmd = TextBoxGetLine(editCommand, 1)
        Rem See what type of parameter to pass in for the input buffer -
        Rem Set according to what radio button is currently set
        If radioAttrType(1).Value = True Then
            Rem Retrieves an Integer type attribute
            nRetOut = nDllRunKlxCmd(pcCmd, 4, nDataIn, nSizeOut, pcErrOut)
        ElseIf radioAttrType(2).Value = True Then
            Rem Retrieves a Float type attribute
            nRetOut = nDllRunKlxCmd(pcCmd, 8, dfDataIn, nSizeOut, pcErrOut)
        Else
            Rem Retrieves a String type attribute
            nRetOut = nDllRunKlxCmd(pcCmd, 1024, ByVal pcDataIn, nSizeOut, pcErrOut)
        End If
        ...
    
    End Sub

## Seamless login tokens and sample code
The seamless login tokens are generated from the host application using the API provided by the vendor.

The following table provides the token description. Some of the tokens come from the XML summary provided by the BSP application. Other tokens come from the host application itself.

| Token            | Value                                                        | Usage in BSP                                                 | Source of the value                                          |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Client           | XXXXXXXXXX                                                   | BSP identifier number for HOST                               | Part of <session> aggregate in the XML summary               |
| Type             | ApplicationMenu                                              | BSP service                                                  | Part of <session> aggregate in the XML summary               |
| Lang             | EN                                                           | Prefered language of the customer                            | Part of <session> aggregate in the XML summary               |
| Userid           | <userid> value as per XML                                    | Primary account for the statement verification               | Part of <session> aggregate in the XML summary               |
| Pswd             | Global password                                              | Authentication of host                                       | Global password common for both application for authentication purposes. |
| sv_HostAppId     | Extension token that identifies the calling application (value to be  supplied by host application) | Audit tracking                                               | Host application                                             |
| sv_HostUserType  | Extension token that identifies whether the user is internal user or  customer(value to be supplied by host application) | May be useful in billing applications at Hosting FI. BSP will treat it as  audit data. | Host application                                             |
| sv_HostUserID    | UserID as per calling application(value to be supplied by host  application) | Audit tracking                                               | Host application                                             |
| sv_HostSessionID | Session ID as per calling application                        | To pass on to Host Image Server To pass back to host to keep the session  alive | Host application                                             |
| sv_ImageItemReqd | N' if image link is not to be shown for the statement. 'Y' is the default  behavior. | BSP hides the links in a flexible way for the calling application. | Host application                                             |
| sv_ReturnURL     | URL to return to if BSP application session ends.            |                                                              | Host application                                             |
| Token Time out   | Time out period for the token built = time out value of host application | Validation of the token.                                     | Host application                                             |

### Sample java code to generate the seamless token

```
/*

 * Created on Oct 22, 2025.
   */
   package com.[vendorname].to.test;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

import com.derivion.api.keys.KeySet;
import com.derivion.api.keys.KeySetLoader;
import com.derivion.api.login.AuthException;
import com.derivion.api.login.AuthToken;
import com.derivion.api.login.Credentials;
import com.derivion.api.provider.FileKeySetProvider;
import com.derivion.api.provider.KeySetProvider;

/**

 * Example code for generation of URL
   */
   public class GenSLExample {

   /**

    * This method initialize the keys.

    * <p>

    * TODO: Replace the file name and passwords with the production values.

    * 

    * @return keyset
      */
      private static KeySet initKeys() throws Exception {
      final String ksFname = "test/fib/FIBSL.ksf";
      final String password1 = "rmu40wCt5EM8TyLvuf6AV";
      final String password2 = "NKXPcS74CpydHd3EofcZW";

      KeySetProvider ksp = new FileKeySetProvider(ksFname);
      KeySetLoader ksl = new KeySetLoader(ksp.getKeySetBytes());
      KeySet ks = ksl.getKeySet(password1, password2);
      return ks;
      }

   /**

    * Generate an HTML sample page.

    * 

    * @param imageUrl - base URL from the XML

    * @param type - service type from XML

    * @param client - client id from XML

    * @param lang - prefered language from XML

    * @param userId - user id from XML

    * @param globalPassword - global password

    * @param properties - list of properties

    * @param channelName - encryption channel name 

    * @param keySet - encryption keys

    * @throws AuthException in case of error
      */
      public static void generateUrl(String imageUrl, String type, String client,
          String lang, String userId, String globalPassword,
          HashMap properties, String channelName, KeySet keySet) throws AuthException {

      int tokenTimeout = 10 * 60; // 10 minutes timeout
      Credentials cr = new Credentials(userId, globalPassword);

      for (Iterator it = properties.entrySet().iterator(); it.hasNext(); ) {
          Map.Entry entry = (Map.Entry) it.next();
          String name = (String) entry.getKey();
          String value = (String) entry.getValue();
          cr.addCredential(name, value);
      }

      // Add to the credentials the HostXXX values
      // TODO: retrieve the HostXXX values
      cr.addCredential("sv_HostAppId", "xxx");
      cr.addCredential("sv_HostUserType", "yyy");
      cr.addCredential("sv_HostUserId", "zzz");
      cr.addCredential("sv_HostSessionId", "zzz");
      cr.addCredential("sv_ImageItemReqd", "Y");
      cr.addCredential("sv_ReturnURL", "http://something");

      // Encrypt the credentials into a token
      AuthToken at = new AuthToken(cr, keySet, channelName, tokenTimeout);

      // Generate an HTML sample page using with an embedded POST request.
      StringBuffer buffer = new StringBuffer();
      buffer.append("<html>\n<body>\n");
      // The base url should come from the <imageurl> tag of the XML
      buffer.append("<form method=\"POST\" action=\"").append(imageUrl)
              .append("\">\n");
      // value comming from the <type> tag from the XML
      buffer.append("<input type=\"hidden\" name=\"type\" value=\"")
              .append(type).append("\">\n");
      // value comming from the <client> tag from the XML
      buffer.append("<input type=\"hidden\" name=\"client\" value=\"")
              .append(client).append("\">\n");
      // value comming from the <lang> tag from the XML
      buffer.append("<input type=\"hidden\" name=\"lang\" value=\"")
              .append(lang).append("\">\n");
      // last piece is the authorization token. Please observe the spelling
      // it is uppercase T.
      buffer.append("<input type=\"hidden\" name=\"authTkn\" value=\"")
              .append(at.getAuthToken()).append("\">\n");

      buffer.append("<input type=\"submit\" value=\"Send request\">\n");
      buffer.append("</body>\n</html>\n");
      System.out.println("HTML sample page:");
      System.out.println(buffer.toString());
      }

   public static void main(String[] args) throws Exception {
       // TODO: replace the initalization with proper retrieval
       // <imageurl> tag
       String imageUrl = "http://localhost:8080/fib/inetSrv";
       // <type> tag
       String type = "ApplicationMenu";
       // <client> tag
       String client = "123456";
       // <lang> tag
       String lang = "EN";
       // the agreed value between the vendor and FI
       String globalPassword = "topsecret";
       // <userid> tag from XML
       String userId = "DP12345";
       

       String channelName = "FIB";
       // keyset should be initialized using the seamless login API
       KeySet keySet = initKeys();
       
       // list of name/value pairs that should be read from the XML
       //  (<property name="XX" value="YY"/> tags from XML)
       HashMap properties = new HashMap();
       properties.put("sv_accountnumber", "DP400399");
       properties.put("sv_pg", "1");
       properties.put("sv_totpg", "1");
       properties.put("sv_billid", "63");
       properties.put("ParmSet", "CSP");
       
       // Generate the URL
       generateUrl(imageUrl, type, client, lang, userId, globalPassword,
               properties, channelName, keySet);

   }
```
