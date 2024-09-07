unit Unit1;

interface

uses
  Winapi.Windows, Winapi.Messages, System.SysUtils, System.Variants,
  System.Classes, Vcl.Graphics,
  Vcl.Controls, Vcl.Forms, EncdDecd, netencoding, Vcl.Dialogs, Vcl.StdCtrls,
  sButton, sMemo,
  CWMIBase, COperatingSystemInfo, idglobal, ipwcore, ipwtypes, ipwjson,
  IdBaseComponent, IdCoder, IdCoder3to4, IdCoderMIME, CryptBase,
  ipccore, ipctypes, ipcaes, FireDAC.UI.Intf, FireDAC.VCLUI.Wait,
  FireDAC.Stan.Intf, FireDAC.Stan.Option, FireDAC.Stan.Param,
  FireDAC.Stan.Error, FireDAC.DatS, FireDAC.Phys.Intf, FireDAC.DApt.Intf,
  FireDAC.Stan.Async, FireDAC.DApt, FireDAC.Stan.Def, FireDAC.Stan.Pool,
  FireDAC.Phys, Vcl.ComCtrls, Data.DB, FireDAC.Comp.Client,
  FireDAC.Comp.DataSet, FireDAC.Comp.UI, FireDAC.Phys.SQLite, sbxcore,
  sbxtypes, sbxsymmetriccrypto, StrUtils, Winapi.Security.Cryptography,
  sSkinManager, System.ImageList, Vcl.ImgList, acAlphaImageList, sListView, AESObj, MiscObj;

type
  TForm1 = class(TForm)
    ipwJSON1: TipwJSON;
    FDGUIxWaitCursor: TFDGUIxWaitCursor;
    FDQuery: TFDQuery;
    FDConnection: TFDConnection;
    AESGCM1: TAESGCM;
    ListView1: TListView;
    Button1: TButton;
    OperatingSystemInfo1: TOperatingSystemInfo;
    procedure FormResize(Sender: TObject);
    procedure Button1Click(Sender: TObject);
  private
    { Private declarations }
  public
    Original_Login_Data, New_Login_Data: String;
    Original_Local_State, New_Local_State: String;
    username: string;
    // Used to grab the username from Microsoft Windows Operating System...
    JSON: TipwJSON;
  end;

var
  Form1: TForm1;

implementation

type
  DATA_BLOB = record
    cbData: DWORD;
    pbData: PBYTE;
  end;

function CryptUnprotectData(pDataIn: PDATA_BLOB; ppszDataDescr: PPWideChar;
  pOptionalEntropy: PDATA_BLOB; pvReserved: Pointer; pPromptStruct: Pointer;
  dwFlags: DWORD; pDataOut: PDATA_BLOB): BOOL; stdcall; external 'Crypt32.dll';

{$R *.dfm}
function dpApiUnprotectData(fpDataIn: TBytes): TBytes;
var
  DataIn, DataOut: DATA_BLOB;
begin
  DataOut.cbData := 0;
  DataOut.pbData := nil;

  DataIn.cbData := Length(fpDataIn);
  DataIn.pbData := @fpDataIn[0];

  if not CryptUnprotectData(@DataIn, nil, nil, nil, nil, 0, @DataOut) then
    RaiseLastOSError;

  SetLength(Result, DataOut.cbData);
  Move(DataOut.pbData^, Result[0], DataOut.cbData);
  LocalFree(HLOCAL(DataOut.pbData));
end;

procedure TForm1.Button1Click(Sender: TObject);
var
  Original_Login_Data, New_Login_Data: String;
  Original_Local_State, New_Local_State: String;
  username: string;
  JSON: TipwJSON;
  Encrypted_Key: string;
  keybytes: tBytes;
  KEY: tBytes;
  password: tBytes;
  ivbytes: tBytes;
  outpass: string;
  LI: Tlistitem;
  DBURL: string;
  DBUSERNAME: string;
begin
  self.ListView1.Clear;
  // Clear out the listview before doing anything in case of previous results...
  form1.OperatingSystemInfo1.Active := true;
  // activating the OS system information component which can tell us the current logged in users "Username"...
  username := Form1.OperatingSystemInfo1.OperatingSystemProperties.RegisteredUser; // Grabbing the current logged in users Username...

  // Get Local State file from the web browsers directory we are going to pull and decrypt passwords from...
  Original_Local_State := 'C:\Users\' + username +
    '\AppData\Local\Google\Chrome\User Data\Local State';
  New_Local_State := 'C:\Users\' + username +
    '\AppData\Local\Temp\Local State.json';

  // check if we already copied a Database file once before and remove it if we find it so we can pull a new one directly from the web browsers install directory in Microsoft Windows...
  if FileExists(New_Local_State) then
    DeleteFile(PChar(New_Local_State));
  CopyFile(PChar(Original_Local_State), PChar(New_Local_State), true);

  // Copy the database file in case web browser locks it and does not allow us to access it.... (we copy it to another location so we can access it)...
  Original_Login_Data := 'C:\Users\' + username +
    '\AppData\Local\Google\Chrome\User Data\Default\Login Data';
  New_Login_Data := 'C:\Users\' + username +
    '\AppData\Local\Temp\Login Data.db';

  if FileExists(New_Login_Data) then
    DeleteFile(PChar(New_Login_Data));
  CopyFile(PChar(Original_Login_Data), PChar(New_Login_Data), true);

  // Create a JSON component in RAM memory...(used to parce json files)....
  JSON := TipwJSON.Create(nil);
  try
    JSON.Reset;
    JSON.InputFile := New_Local_State;
    // setup the newly created Json parcer and tell it to look into the browsers json file responsible for storing the "Master Encrypted KEY" which later is used to decrypt saved passwords...
    JSON.Parse;

    JSON.XPath := '/json/os_crypt/encrypted_key';
    Encrypted_Key := JSON.XText;
    // Grabbing the Encrypted Master key from the Json file as "String Text" ....
    Encrypted_Key := StringReplace(Encrypted_Key, '"', '',
      [rfReplaceAll, rfIgnoreCase]);
    // Removing the beginning and ending quotes ""'s from the Encrypted Master KEY .... we do this because we dont want it added to the string later in the program. (For Decrypting purposes)...
  finally
    JSON.Free; // Free up memory by releasing the Json object...
  end;

  // Put the envcrypted key into an array of Bytes...
  keybytes := TNetEncoding.Base64.DecodeStringToBytes(Encrypted_Key);


  // Delete DPAPI signature from the beginning of the Encrypted Master key Bytes... (we must do this in order to use the win32 Unprotect data API function in Microsoft Windows)....
  delete(keybytes, 0, 5);


  KEY := dpApiUnprotectData(keybytes);
  // Decrypt the Master encrypted KEY using the windows API function known as win32 Unprotect Data... it will return the unencrypted Bytes....
  // Setup a Database Connection and connect to the browsers SQlite database we coppied in previous steps...
  Form1.FDConnection.Params.Clear;
  Form1.FDConnection.Params.DriverID := 'SQLite';
  Form1.FDConnection.Params.Database := New_Login_Data;
  Form1.FDQuery.SQL.Clear;
  Form1.FDQuery.SQL.Text :=
    'SELECT origin_url, action_url, username_value, password_value, date_created FROM logins';
  // SQL query to execute grabs the rows containing URL's Usernames, and Encrypted Passwords from the "Logins" table...
  Form1.FDQuery.Open();
  // Open or Execute the SQL Statement so we can begin itterating over the returned Rows from the "Logins" table...

  // Loop through the database table and grab all 3 pieces of information (URL,Username,EncryptedPassword)
  while not Form1.FDQuery.Eof do
  begin
    // clear out all used varables before the next itteration over the next Database table Row...
    DBUSERNAME := '';
    DBURL := '';
    // ensureing that the password is NIL in bytes before we begin the loop each time we itterate over a new item in the database...
    password := nil;

    // Grab the Encrypted Password as Bytes...
    password := Form1.FDQuery.FieldByName('password_value').AsBytes;


    // grabs the URL from the database....
    DBURL := Form1.FDQuery.FieldByName('origin_url').AsString;


    // grabs the Username from the database....
    DBUSERNAME := Form1.FDQuery.FieldByName('username_value').AsString;


    delete(password, 0, 3);
    // Removing the V10 Signature from the beginning of the password...
    ivbytes := copy(password, 0, 12);
    // Copying the (IV): Initilization Vector from the first 12 bytes of the encrypted password...
    delete(password, 0, 12);
    // Removing the (IV): Initilization Vector Bytes from the beginning of the encrypted password...

    // Decrypt our key and display it in PlainText on Screen...
    self.AESGCM1.IVLength := 12;
    // setting the IV's Length which in this case for decrypting chromium based web browsers is a static IV length of 12
    self.AESGCM1.IV := TEncoding.ANSI.GetString(ivbytes);
    // setting the AESGCM Decryptor components IV setting as Bytes...
    self.AESGCM1.KEY := TEncoding.ANSI.GetString(KEY);
    // setting the AESGCM Decryptor components KEY setting as Bytes...
    try
      self.AESGCM1.DecryptAndVerify(TEncoding.UTF7.GetString(password),'',
        outpass); // Starting the process of decrypting the Encrypted password grabbed from the Database...(from the ROW we are currently itteratting over)...


      // Begin the process of creating list items and displaying the results to the user...
    LI := self.ListView1.items.Add;
    LI.Caption := DBURL;
    LI.SubItems.Add(DBUSERNAME);
    LI.SubItems.Add(outpass);
    li.ImageIndex:=0; //0 is for Chromes Icon index...
    // Proceed itterating over the next returned ROW so we can grab the next password ect... ect...
    self.FDQuery.Next;
    Except
      on E: Exception do
      begin
        self.FDQuery.Next;
      end;

    end;

  end;
end;

procedure TForm1.FormResize(Sender: TObject);
begin
Form1.ListView1.Repaint;
end;


end.
