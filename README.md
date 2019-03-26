# Bell Vision

あらかじめ配布されている

- Client ID
- Client Secret
- Refresh Token  

を用いてAccess Tokenの発行を行う  
※Access Tokenの有効期限は発行してから約1時間です。

## Access Tokenの発行

    URL     : http://{HOST}/{PREFIX}/oauth/v1/token
    Methods : POST
    Headers : Content-Type: application/json
    Body    :  
      - client_id  
      - client_secret
      - refresh_token
      - grant_type : 'refresh_token'
    Response
      - access_token
      - expires_in
      - scope
      - token_type


## Example

### vb.net

```vb

Dim PostDataDic As New Dictionary(Of String, String) From
 {
    {"client_id", "{{client_id}}"},
    {"client_secret", "{{client_secret}}"},
    {"refresh_token", "{{refresh_token}}"},
    {"grant_type", "refresh_token"}
 }
Dim PostData As String = JsonConvert.SerializeObject(PostDataDic)
Dim Req As HttpWebRequest = DirectCast(WebRequest.Create("{{URL}}"), HttpWebRequest)
Req.Method = "POST"
Req.ContentType = "application/json"
Dim byteArray As Byte() = Encoding.UTF8.GetBytes(PostData)
Dim DataStream As Stream = Req.GetRequestStream()
DataStream.Write(byteArray, 0, byteArray.Length)
DataStream.Close()
Dim Res As HttpWebResponse = DirectCast(Req.GetResponse(), HttpWebResponse)
Dim ResStream As Stream = Res.GetResponseStream()
Dim ResRead As StreamReader = New StreamReader(ResStream, Encoding.UTF8)
Dim JsonData As String = ResRead.ReadToEnd()
Dim ResData As Dictionary(Of String, String) = JsonConvert.DeserializeObject(Of Dictionary(Of String, String))(JsonData)
Dim Access_Token As String = ResData("access_token")

```

### C#

```C#
Dictionary<string, string> PostDataDic = new Dictionary<string, string>()
{
    {
        "client_id",
        "{{client_id}}"
    },
    {
        "client_secret",
        "{{client_secret}}"
    },
    {
        "refresh_token",
        "{{refresh_token}}"
    },
    {
        "grant_type",
        "refresh_token"
    }
};
string PostData = JsonConvert.SerializeObject(PostDataDic);
HttpWebRequest Req = (HttpWebRequest)WebRequest.Create("{{URL}}");
Req.Method = "POST";
Req.ContentType = "application/json";
byte[] byteArray = Encoding.UTF8.GetBytes(PostData);
Stream DataStream = Req.GetRequestStream();
DataStream.Write(byteArray, 0, byteArray.Length);
DataStream.Close();
HttpWebResponse Res = (HttpWebResponse)Req.GetResponse();
Stream ResStream = Res.GetResponseStream();
StreamReader ResRead = new StreamReader(ResStream, Encoding.UTF8);
string JsonData = ResRead.ReadToEnd();
Dictionary<string, string> ResData = JsonConvert.DeserializeObject<Dictionary<string, string>>(JsonData);
string Access_Token = ResData["access_token"];

```

## 画像認識APIの呼び出し

### Example(共通関数)

### vb.net

```vb
'共有関数
Dim PostData As New StringBuilder()
Dim Bytes As List(Of Byte()) = New List(Of Byte())
Dim ReadData(&H1000) As Byte
Dim ReadSize As Integer = 0
For Each path As String In Files
    Dim FileName As String = IO.Path.GetFileName(path)
    PostData.AppendLine("--" + Boundary)
    PostData.Append($"Content-Disposition: form-data; name=""{name}""; filename=""")
    PostData.AppendLine(FileName + """")
    PostData.AppendLine("Content-Type: application/octet-stream")
    PostData.Append("Content-Transfer-Encoding: binary" + vbCrLf + vbCrLf)
    Using ms As New MemoryStream()
        Dim post_byte() As Byte = Enc.GetBytes(PostData.ToString())
        ms.Write(post_byte, 0, post_byte.Length)
        Using fs As New FileStream(path, FileMode.Open, FileAccess.Read)
            While True
                ReadSize = fs.Read(ReadData, 0, ReadData.Length)
                If ReadSize = 0 Then
                    Exit While
                End If
                ms.Write(ReadData, 0, ReadSize)
            End While
        End Using
        Bytes.Add(ms.ToArray())
    End Using
Next
Return Bytes

'Request & Response
Dim Enc As Encoding = Encoding.UTF8
Dim Boundary As String = System.Environment.TickCount.ToString()
Dim FileLen As Integer = 0
Dim Req As HttpWebRequest = DirectCast(WebRequest.Create(Url), HttpWebRequest)
Req.Headers.Add("Authorization", "Bearer {Token}")
Req.Method = "POST"
Req.ContentType = "multipart/form-data; boundary=" + Boundary
Dim PostData As String = ""
Dim StartData As Byte() = PostDatas(Boundary, FileList, Enc, name).SelectMany(Function(x) x).ToArray()
PostData = vbCrLf + "--" + Boundary + "--" + vbCrLf
Dim EndData As Byte() = Enc.GetBytes(PostData)
Req.ContentLength = StartData.Length + EndData.Length
Dim ReqStream As System.IO.Stream = Req.GetRequestStream()
ReqStream.Write(StartData, 0, StartData.Length)
ReqStream.Write(EndData, 0, EndData.Length)
ReqStream.Close()
Dim Res As HttpWebResponse = DirectCast(Req.GetResponse(), HttpWebResponse)
Dim ResStream As Stream = Res.GetResponseStream()
Using sr As New StreamReader(ResStream, Enc)
    Return sr.ReadToEnd()
End Using

```

### C#

```C#
// 共有関数
StringBuilder PostData = new StringBuilder();
List<byte[]> Bytes = new List<byte[]>();
byte[] ReadData = new byte[4097];
int ReadSize = 0;
foreach (string path in Files)
{
    string FileName = System.IO.Path.GetFileName(path);
    PostData.AppendLine("--" + Boundary);
    PostData.Append($"Content-Disposition: form-data; name=""{name}""; filename=""");
    PostData.AppendLine(FileName + "\"");
    PostData.AppendLine("Content-Type: application/octet-stream");
    PostData.Append("Content-Transfer-Encoding: binary" + Constants.vbCrLf + Constants.vbCrLf);
    using (MemoryStream ms = new MemoryStream())
    {
        byte[] post_byte = Enc.GetBytes(PostData.ToString());
        ms.Write(post_byte, 0, post_byte.Length);
        using (FileStream fs = new FileStream(path, FileMode.Open, FileAccess.Read))
        {
            while (true)
            {
                ReadSize = fs.Read(ReadData, 0, ReadData.Length);
                if (ReadSize == 0)
                    break;
                ms.Write(ReadData, 0, ReadSize);
            }
        }
        Bytes.Add(ms.ToArray());
    }
}
return Bytes;

// Request & Response
Encoding Enc = Encoding.UTF8;
string Boundary = System.Environment.TickCount.ToString();
int FileLen = 0;
HttpWebRequest Req = (HttpWebRequest)WebRequest.Create(Url);
Req.Headers.Add("Authorization", $"Bearer {Token}");
Req.Method = "POST";
Req.ContentType = "multipart/form-data; boundary=" + Boundary;
string PostData = "";
byte[] StartData = PostDatas(Boundary, FileList, Enc, name).SelectMany(x => x).ToArray();
PostData = Constants.vbCrLf + "--" + Boundary + "--" + Constants.vbCrLf;
byte[] EndData = Enc.GetBytes(PostData);
Req.ContentLength = StartData.Length + EndData.Length;
System.IO.Stream ReqStream = Req.GetRequestStream();
ReqStream.Write(StartData, 0, StartData.Length);
ReqStream.Write(EndData, 0, EndData.Length);
ReqStream.Close();
HttpWebResponse Res = (HttpWebResponse)Req.GetResponse();
Stream ResStream = Res.GetResponseStream();
using (StreamReader sr = new StreamReader(ResStream, Enc))
{
    return sr.ReadToEnd();
}

```

## Single Recognition

    URL     : http://{HOST}/{PREFIX}/api/v1/single_recognition
    Methods : POST  
    Headers : 
      - Content-Type: multipart/form-data  
      - Authorization: Bearer {access_token}
    Body    :  
      - file  
    Response
     - images
        - classes: []
          - class
          - score
        - bestclass
          - class
          - index
          - score

## Example

### vb.net

```vb

Sub Single_Recognition()
    Dim FilePath As New List(Of String) From {{"{{FilePath}}"}}
    Dim Url As String = "http://{HOST}/{PREFIX}/api/v1/single_recognition"
    Console.WriteLine(SendData(Url, FilePath, "file"))
End Sub

```

### C#

```C#

public void Single_Recognition()
{
    List<string> FilePath = new List<string>() { { "{{FilePath}}" } };
    string Url = "http://{HOST}/{PREFIX}/api/v1/single_recognition";
    Console.WriteLine(SendData(Url, FilePath, "file"));
}

```

## Multi Recognition

    URL     : http://{HOST}/{PREFIX}/api/v1/multi_recognition
    Methods : POST  
    Headers : 
      - Content-Type: multipart/form-data  
      - Authorization: Bearer {access_token}
    Body    :  
      - file: []  
    Response
     - images: []
        - classes: []
          - class
          - score
        - bestclass
          - class
          - index
          - score

## Example

### vb.net

```vb

Sub Maluti_Recognition()
    Dim FilePath As List(Of String) = New List(Of String) From
        {
            {"{{FilePath}}"},
            {"{{FilePath}}"}
        }
    Dim Url As String = "http://{HOST}/{PREFIX}/api/v1/multi_recognition"
    Console.WriteLine(SendData(Url, FilePath, "files"))
End Sub

```

### C#

```C#

public void Maluti_Recognition()
{
    List<string> FilePath = new List<string>()
    {
        {
            "{{FilePath}}"
        },
        {
            "{{FilePath}}"
        }
    };
    string Url = "http://{HOST}/{PREFIX}/api/v1/multi_recognition";
    Console.WriteLine(SendData(Url, FilePath, "files"));
}

```
