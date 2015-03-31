#Send SMS in Java

This tutorial will walk you through building a simple Java project that uses Sinch to sign a request to [send an SMS](https://www.sinch.com/products/sms-api/). Why sign your requests instead of using basic authentication? It's much more secure, and Sinch doesn't support basic authentication for production applications. 

You can find the finished project on our Github at [github.com/sinch/java-sms](http://www.github.com/sinch/java-sign-requests)

##Setup

If you don't yet have a Sinch developer account, create one at [sinch.com/signup](https://www.sinch.com/dashboard/#/signup). Use the developer dashboard to create an app (you only need to input an app name), and take note of the app key and secret. 

##Library for Base64 Encoding

You will need to download and add the commons codec library to your Eclipse project to encode your app key and secret. [http://commons.apache.org/proper/commons-codec/download_codec.cgi](http://commons.apache.org/proper/commons-codec/download_codec.cgi). In your project, create a **libs** folder at the same level as **src**, and move **commons-codec-1.10.jar** to this folder. Then, right-click the .jar file and click **build path --> add to build path**

##Send an SMS

Start by creating a new class - I'll name mine **Sms.java**. To keep things simple, I put the code to send an SMS in the **main** function. To make this work for you, substitute in a phone number (in [E.164 format](http://en.wikipedia.org/wiki/E.164)) to send the message to, your app key, and your app secret. 

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;
	import java.io.OutputStream;
	import java.net.HttpURLConnection;
	import java.net.URL;
	
	import org.apache.commons.codec.binary.Base64;
	import java.util.Date;
	import java.util.TimeZone;
	import java.text.SimpleDateFormat;
	import java.text.DateFormat;
	import java.security.MessageDigest;
	
	import javax.crypto.Mac;
	import javax.crypto.spec.SecretKeySpec;
	
	public class Sms {
		
		public static void main(String[] args) {
			
			try {
				String to = "phone_number";
				String key = "your_app_key";
				String secret = "your_app_secret";
				String message = "Hello, world!";
				
				Date date= new java.util.Date();
				DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
				dateFormat.setTimeZone(TimeZone.getTimeZone("UTC"));
				String timestamp = dateFormat.format(date);
				
				String httpVerb = "POST";
				String path = "/v1/sms/" + to;
				String contentType = "application/json";
				String canonicalizedHeaders = "x-timestamp:" + timestamp;			
				String body = "{\"message\":\"" + message + "\"}";
				
				String contentMd5 = Base64.encodeBase64String(md5Digest(body));
				String stringToSign = httpVerb + "\n" + contentMd5 + "\n" + contentType + "\n" + canonicalizedHeaders + "\n" + path;
				String signature = signature(secret, stringToSign);
				String authorization = "Application " + key + ":" + signature;
				
	            URL url = new URL("https://messagingApi.sinch.com" + path);
	            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
	            connection.setDoOutput(true);
	            connection.setRequestMethod("POST");
	            connection.setRequestProperty("content-type", "application/json");
	            connection.setRequestProperty("x-timestamp", timestamp);
	            connection.setRequestProperty("authorization", authorization);
	            
	            OutputStream os = connection.getOutputStream();
	            os.write(body.getBytes());
	            
	            StringBuilder response = new StringBuilder();
	            BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
	            
	            String line;
	            while ( (line = br.readLine()) != null)
	                response.append(line);
	            
	            br.close();
	            os.close();
	            
	            System.out.println(response.toString());
	
	        } catch (IOException e) {
	            e.printStackTrace();
	        }		
		}
		
		private static byte[] md5Digest(String body) {
			MessageDigest md = null;		
			byte[] bytesOfBody = null;
			
			try {
				bytesOfBody = body.getBytes("UTF-8");
				md = MessageDigest.getInstance("MD5");
			} catch (Exception e) {
				e.printStackTrace();
			}
	
			return md.digest(bytesOfBody);
		}
		
		private static String signature(String secret, String message) {		
			String signature = "";
			
			try {			
			     Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
			     SecretKeySpec secret_key = new SecretKeySpec(Base64.decodeBase64(secret.getBytes()), "HmacSHA256");
			     sha256_HMAC.init(secret_key);
			     signature = Base64.encodeBase64String(sha256_HMAC.doFinal(message.getBytes()));
			} catch (Exception e){
				System.out.println("Error");
			}
			
			return signature;
		}
	
	}
	
	
That's all! Run the project, and you will see "Hello, world!" appear as an SMS to the phone number you entered. 

In addition, you can find SMS prices by country here: [sinch.com/pricing/sms-prices](https://www.sinch.com/pricing/sms-prices/)
	
