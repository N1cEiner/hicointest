
3 Appendix
==========

Attachment 1:Encryption and Decryption Mode
~~~~~~~~~~~~~~~~~~~~~~~~

The values of the request parameter DATA and the response field DATA are both encrypted by RSA and encrypted by **base64urlsafe**

*Matters needing attention*

1）There are two symbols + and/that are escaped directly by URL in Base64 traditional encoding. Therefore, if we want to transfer these encoded strings through URL, we need to do traditional Base64 encoding first, then replace + and/with two characters - _ respectively, and do the opposite action decoding at the receiving end

2) Java Demo ： https://github.com/HiCoinCom/WaaSDemo

3）RSA encryption and decryption using segmented encryption

:Example of request parameter encryption:

::

	 // Raw request parameters
	 String originReqData = '{"charset":"utf-8","symbol":"eth","sign":"","time":"1586420916306","app_id":"baaceb1e506e1b5d7d1f0a3b1622583b","version":"2.0"}'

	 // The encryptByPrivate method is encapsulated in the following public class, RSAHelper.java
	 String encryptReqData = RSAHelper.encryptByPrivate(originReqData, "third party its own private key")

	 //http post
	 String httpBuildParams = "app_id=baaceb1e506e1b5d7d1f0a3b1622583b&data=" + encryptReqData



:Response data decryption example:

::

	// Raw data for the response
  String originResp= '{"data":"jwtkGrhh2EVJS8xe93MpUYd-SQ-TyK0Bx5sXjE4hygFNg4wmctiahtIYXRpR2j8yDaEF5YzVstnUKbOH2p44FSMjXMQU4qFrhD00WOfW7v4LNALyiQXRb_5sakR0Zf573lGfLRTPlzLtTho3gqu3hMwuAv5e3r2dpb6_jxh1Z9BjkzSsNRX_bjLcHLUOPhMvo6rTUKSa9LQ6QnT8RX0eqzOZPlnCw3TeX_zcWWjxp6fcpKcdODxoI86gHwWRpSd-2qbEbFcaT12CJd9nPXA0KnLPNNHWz8sxQGiAg7Jg_-cN_yBHL9cS15zecTemYGqpOXRkojM1JwLsjM-7txf_dw"}'

	// Decrypt response data
	String encryptRespData = JSON.parse(originResp)['data']
	// decryptByPublic is encapsulated in the following public class RSAHelper.java
  String decryptRespData = RSAHelper.decryptByPublic( encryptRespData, "hosting platform provides the public key" )


:Public class RSAHelper.java:

::


	import java.io.ByteArrayOutputStream;
	import java.security.Key;
	import java.security.KeyFactory;
	import java.security.spec.PKCS8EncodedKeySpec;
	import java.security.spec.X509EncodedKeySpec;
	import java.util.Base64;
	import java.util.Base64.Decoder;
	import java.util.Base64.Encoder;

	import javax.crypto.Cipher;

	public class RSAHelper {
		/**
	     * Encryption algorithm RSA
	     */
	    public static final String KEY_ALGORITHM = "RSA";

	    /** *//**
	     * RSA maximum encryption plaintext size
	     */
	    private static final int MAX_ENCRYPT_BLOCK = 234;

	    /** *//**
	     * RSA maximum decrypted ciphertext size
	     */
	    private static final int MAX_DECRYPT_BLOCK = 256;


	    private static final String CHARSET ="UTF-8";



	    /**
	     * Public key decryption
	     *
	     * @param encryptedData encrypted data
	     * @param publicKey publicKey (Base64 encoding)
	     * @return
	     * @throws Exception
	     */
	    public static byte[] decryptByPublicKey(byte[] encryptedData, String publicKey)
	                    throws Exception {
	            byte[] keyBytes =  decryptBASE64(publicKey);
	            X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(keyBytes);
	            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
	            Key publicK = keyFactory.generatePublic(x509KeySpec);
	            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
	            cipher.init(Cipher.DECRYPT_MODE, publicK);
	            int inputLen = encryptedData.length;
	            ByteArrayOutputStream out = new ByteArrayOutputStream();
	            int offSet = 0;
	            byte[] cache;
	            int i = 0;
	            // Decrypt the data piecewise
	            while (inputLen - offSet > 0) {
	                    if (inputLen - offSet > MAX_DECRYPT_BLOCK) {
	                            cache = cipher.doFinal(encryptedData, offSet, MAX_DECRYPT_BLOCK);
	                    } else {
	                            cache = cipher.doFinal(encryptedData, offSet, inputLen - offSet);
	                    }
	                    out.write(cache, 0, cache.length);
	                    i++;
	                    offSet = i * MAX_DECRYPT_BLOCK;
	            }
	            byte[] decryptedData = out.toByteArray();
	            out.close();
	            return decryptedData;
	    }

	    /**
	     *  Public key segment decryption
	     * @param encryptedData encrypts base64 data
	     * @param publicKey rsa Public Key
	     * @return
	     */
	    public static String decryptByPublicKey(String encryptedData, String publicKey){
	            if(encryptedData==null || encryptedData.isEmpty() || publicKey==null || publicKey.isEmpty()) {
	            	return "";
	            }

	            try {
	                encryptedData = encryptedData.replace("\r", "").replace("\n", "");
	                byte[] data = decryptByPublicKey(decryptBASE64(encryptedData), publicKey);
	                if(data == null || data.length < 1){
	                        return  "";
	                }
	                return new String(data);
	            }catch (Exception ex){
	                    ex.printStackTrace();
	            }
	            return "";
	    }

	    /**
	     * Private key encryption
	     *
	     * @param data source data
	     * @param privateKey (BASE64 encode)
	     * @return
	     * @throws Exception
	     */
	    public static byte[] encryptByPrivateKey(byte[] data, String privateKey)
	                    throws Exception {
	            byte[] keyBytes =  decryptBASE64(privateKey);
	            PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(keyBytes);
	            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
	            Key privateK = keyFactory.generatePrivate(pkcs8KeySpec);
	            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
	            cipher.init(Cipher.ENCRYPT_MODE, privateK);
	            int inputLen = data.length;
	            ByteArrayOutputStream out = new ByteArrayOutputStream();
	            int offSet = 0;
	            byte[] cache;
	            int i = 0;
	            // Encrypt the data in segments
	            while (inputLen - offSet > 0) {
	                    if (inputLen - offSet > MAX_ENCRYPT_BLOCK) {
	                            cache = cipher.doFinal(data, offSet, MAX_ENCRYPT_BLOCK);
	                    } else {
	                            cache = cipher.doFinal(data, offSet, inputLen - offSet);
	                    }
	                    out.write(cache, 0, cache.length);
	                    i++;
	                    offSet = i * MAX_ENCRYPT_BLOCK;
	            }
	            byte[] encryptedData = out.toByteArray();
	            out.close();
	            return encryptedData;
	    }

	    /**
	     *  The private key segments the data
	     * @param data Data to be encrypted
	     * @param privateKey
	     * @return
	     */
	    public static String encryptByPrivateKey(String data, String privateKey){
	            if(data==null || privateKey==null || data.isEmpty()|| privateKey.isEmpty()) {
	            	return "";
	            }

	            try {
	                    byte[] encryptedData = encryptByPrivateKey(data.getBytes(CHARSET), privateKey);
	                    if(encryptedData == null || encryptedData.length < 1){
	                            return  "";
	                    }

	        byte[] dataBytes = encryptBASE64(encryptedData).getBytes(CHARSET);
	        return new String(dataBytes).replace("\r", "").replace("\n", "");
	            }catch (Exception ex){
	                    ex.printStackTrace();
	            }
	            return "";
	    }

	    /**
	     * BASE64Encoder encryption
	     *
	     * @param data
	     *            Data to encrypt
	     * @return encrypted string
	     */
	    public static String encryptBASE64(byte[] data) {
	    	//Under the JDK 1.8 environment, use the following 2 lines of code
	        // BASE64Encoder encoder = new BASE64Encoder();
	        // String encode = encoder.encode(data);
	        // The rt.jar package has been deprecated since JKD 9. Java.util.base64.encoder has been used since JDK 1.8
	        Encoder encoder = Base64.getEncoder();
	        String encode = encoder.encodeToString(data);
	        //No matter what environment you are using, the following +/ is replaced with -_
	        String safeBase64Str = encode.replace('+', '-');
	        safeBase64Str = safeBase64Str.replace('/', '_');
	        safeBase64Str = safeBase64Str.replaceAll("=", "");
	        return safeBase64Str;
	    }
	    /**
	     * BASE64Decoder decryption
	     *
	     * @param data
	     *            The string to decrypt
	     * @return decrypted byte[]
	     * @throws Exception
	     */
	    public static byte[] decryptBASE64(String data) throws Exception {
	    	// Under the JDK 1.8 environment, use the following 2 lines of code
	        // BASE64Decoder decoder = new BASE64Decoder();
	        // byte[] buffer = decoder.decodeBuffer(data);
	        // The rt.jar package has been deprecated from JKD 9. Java.util.base64.decoder has been used from JDK 1.8
	        Decoder decoder = Base64.getDecoder();

	        //Regardless of the environment used, the following substitution of -_ with +/ needs to be done.
	        String base64Str = data.replace('-', '+');
	        base64Str = base64Str.replace('_', '/');
	        int mod4 = base64Str.length()%4;
	        if(mod4 > 0){
	            base64Str = base64Str + "====".substring(mod4);
	        }

	        byte[] buffer = decoder.decode(base64Str);
	        return buffer;
	    }
	}


:PHP Demo:

::

	<?php
		class RSA
		{

			//Third party private key
		    public $pri_key = 'MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQD6YNILWOJZjS6FQQ9ZL9CEKcWZTTldrDLsxP2dQME7hSUTDQ5AosBUZk18Uq212SC2+L0UA9G6WPoCNzHCB8TP25jC+EwIkHMN4EEPRs+bEHUgX3Bq3oR2SCHjEiqleTFW2kO/oS6Vg9bhTST5MFaEnA0fc2Bh3+4iRus+5mVc6ux0lG55f1qmvUNM4hhP7qVpzc3X0xFA0Slu8dyel1dbOUQlJbUkrt5NzXXqmRoP5UVHUCXPZzH1kbxdbGA58TonXceh6DHQRa6pIBNaQ6BfnqhMvGVvuIqKPrdWq8yigvTw2zqBfwCwY3/3FZoI5ICQ8oS3GRHYP/rXzncqkKTzAgMBAAECggEAdag77EMnkueKXeo12TZj6Udr6N9mPsOl5qenelcsttiZlHtFIFCays6MSQjdQqA3BGSdDaPB0azwR0xCoKhf70GFZtGhgUDIIFQqnpArDPZN5BmVTVMlsiOxcPBfhAUQj3zf61RF/NLIjnVfE46IiaZ/cDEasMO3NvpWn+dK6L86zklgwHfC5IXTFnFRVA3bWkAQ3gswhLzjs50HNoNV96fsnbt1n7NSWhyz9B8hGMV+qYz1NGmb+VsaAune+oIv28krcaqf+Doah37rCmzEgVeZZ1/flPFOXpaq1eGJDgbLu6FbbgqfabCBlhmuzuwNbDl/2T/U9U6JoQWGR7t++QKBgQD8XSzBqpWwz8ebfsPipvnhIugbHgBnwLaRc3/xieuNuiDMsYPY1isBWSeYqjwV5uTad9s9dRxb6OOMH+KChkUxkYhEvoujUulGSuO4MxJlWl99WWEsbLzefubBD0zyHo5daHbPPXO8UPMu/SfiYxT2D5wsW2/swUqHWS3AmDS9RQKBgQD9/FJC/++DLyhU60Q9vrVY50zQTyPLtPnuIxbsPXB1Exo1wKe+LC02k9Cub9f5EFViTEniWRasB7ecnDxJT/ISU+hJjMUKFuaHueb7dO6wiIqyfpJeQM/4fKalBQI+nCEh3aceNKP44mk/lv2x22+P47EAKh7yqBdEVUv5GlHw1wKBgQCbAqReJOijXU0vLtMlYgj0h9tn5Kq9D/tUJky9UUkVmfFRqevhgdOSlW+j71TO4y9JHfvVqRyNO+ShCmi4Yb8Yrlq0VxIwdNoCqjdryjsPdE5ZEVCF2Bi+1dXpWfuacLhjman4q7duQY7OGwOno9KZPYdhG50JIMUlk9pthVBHvQKBgCXUC+iAuAqg3m/vboWHvvjT0mQANYOkm8j1HvfmmrZFNxUkcZdoev9y+pTQgalN3nm6hRKaVD8hEx7XQj9lEdfa+XDi74H2MTWr4ZQ4MUjHvWiiY2h4XMFUx3kyisgKdwDVQ4vDKVzrU+OtuHFiDnau4fD1VRCtKnH6Bku+uM+XAoGAB7V/OFlk7gaX7gne7p+DypXICn1oGE46aFLsDciOyePNovYg6bfdiUB9evwFSijiHq7eldZIQSRIdUalL1qfv2zDwFmEGpSd/RZYOOv21c3eISjln6W7ZGtumtLHx2nGpC072i5vNee0aAPEdvO0h3y4gvzad5L4KwIwyHifKic=';

			//The public key given by the wallet
		    public $pub_key = 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAua4XMw/W9BxyZhirTlNau5Y/tdAHkPsbZo58Cdz1ByeRX8RwOibpREDZLTwhMTZGroqWEAZ+efQhx0gez++03Sw6IsPWPDpzpM90ezn2gBqPog7jxQA+M0E32gMHWB5ygplPwQkGz/qGYeJ5qhp2OZ8O+jFqOJNi7ob1hE2QsPT118HIhUzTL77urD61BovI+jg9Rx6PGAqlFLLmfXToqDulLkYVKhhQlL7ii6iuzIXgl46mbmvH2RXJRq083sa9b9J1z/WzXxNaHNpq5USl3ifTTyD/IiOKnblA7f4KJmr9rcMFbAP1mNxz95at6hPBvqGypPqqixxPBrdkOIPUVwIDAQAB';

			public function __construct() {

		        if (!is_null($this->pub_key)) {
		            $this->pub_key = openssl_pkey_get_public($this->formatterPublicKey($this->pub_key));
		        }

		        if (!is_null($this->pri_key)) {
		            $this->pri_key = openssl_pkey_get_private($this->formatterPrivateKey($this->pri_key));
		        }

		    }


			 /**
		     * Formats the public key
		     * @param $publicKey string 
		     * @return string
		     */
		    public function formatterPublicKey($publicKey)
		    {
		        if (false !== strpos($publicKey, '-----BEGIN PUBLIC KEY-----')) return $publicKey;

		        $str = chunk_split($publicKey, 64, PHP_EOL);//Add a \n after each 64 character
		        $publicKey = "-----BEGIN PUBLIC KEY-----".PHP_EOL.$str."-----END PUBLIC KEY-----";

		        return $publicKey;
		    }

		    /**
		     * Formats the private key
		     * @param $privateKey string
		     * @return string
		     */
		    public function formatterPrivateKey($privateKey)
		    {
		        if (false !==strpos($privateKey, '-----BEGIN RSA PRIVATE KEY-----')) return $privateKey;

		        $str = chunk_split($privateKey, 64, PHP_EOL);//Add a \n after each 64 character
		        $privateKey = "-----BEGIN RSA PRIVATE KEY-----".PHP_EOL.$str."-----END RSA PRIVATE KEY-----";

		        return $privateKey;
		    }

			/**
		     * URL base64 decoded
		     * '-' -> '+'
		     * '_' -> '/'
		     * Remaining number of string length %4, complement '='
		     * @param unknown $string
		     */
		  function urlsafe_b64decode($string) {
		        $data = str_replace(array('-','_'),array('+','/'),$string);
		        $mod4 = strlen($data) % 4;
		        if ($mod4) {
		            $data .= substr('====', $mod4);
		        }
		        return base64_decode($data);
		    }

		    /**
		     * URL base64 encoding
		     * '+' -> '-'
		     * '/' -> '_'
		     * '=' -> ''
		     * @param unknown $string
		     */
		    function urlsafe_b64encode($string) {
		        $data = base64_encode($string);
		        $data = str_replace(array('+','/','='),array('-','_',''),$data);
		        return $data;
		    }


		    /**
		     *  private key encryption (segment encryption)
		     *  emptyStr    requires an encrypted string
		     */
		    public function encrypt($str) {
		        $crypted = array();
		//        $data = json_encode($str);
		        $data = $str;
		        $dataArray = str_split($data, 234);
		        foreach($dataArray as $subData){
		            $subCrypted = null;
		            openssl_private_encrypt($subData, $subCrypted, $this->pri_key);
		            $crypted[] = $subCrypted;
		        }
		        $crypted = implode('',$crypted);
		        return $this->urlsafe_b64encode($crypted);
		    }

		    /**
		     *  Public key decryption (segmented decryption)
		     *  @encrypstr
		     */
		    public function decrypt($encryptstr) {
		        // echo $encryptstr;exit;
		        $encryptstr = $this->urlsafe_b64decode($encryptstr);
		        $decrypted = array();
		        $dataArray = str_split($encryptstr, 256);

		        foreach($dataArray as $subData){
		            $subDecrypted = null;
		            openssl_public_decrypt($subData, $subDecrypted, $this->pub_key);
		            $decrypted[] = $subDecrypted;
		        }
		        $decrypted = implode('',$decrypted);
		        // openssl_public_decrypt(base64_decode($encryptstr),$decryptstr,$this->pub_key);
		        return $decrypted;
		    }

		}
		$rsa = new RSA();
		$params = '{"charset":"utf-8","country":"+86","sign":"","mobile":"","time":"1589013592078","app_id":"baaceb1e506e1b5d7d1f0a3b1622583b","version":"2.0","email":"test123@qq.com"}';

		//Encryption parameters
		$encryptParamsByPriv = $rsa->encrypt($params);

		//Request Interface
		$resp = file_get_contents('http://awstestopenapi.hicoin.one/api/v2/user/info?app_id=baaceb1e506e1b5d7d1f0a3b1622583b&data='.$encryptParamsByPriv);

		$resp = json_decode($resp, true)['data'];
		//Decryption interface returns
		echo "get user info api:", $rsa->decrypt($resp);







Attachment  2:Interface error code list
~~~~~~~~~~~~~~~~~~~~~~~~
========  ==================================================================
code      msg
0         success
100001	  system error
100004	  request parameter is not valid
100005	  signature verification failed
100007	  illegal IP
100015	  merchant ID is invalid
100016	  merchant information expired
110004	  users are frozen and cannot withdraw money
110023	  mobile phone number has been registered
110037    withdrawal address is at risk
110055	  correct withdrawal address for
110065	  user does not exist (used to obtain user balance, withdraw money, or transfer money)
110078	  withdrawal or transfer amount less than the minimum transfer amount
110087	  withdrawal or transfer amount is greater than the maximum transfer amount
110088	  please do not repeat the request
110089	  registered mobile phone number is incorrect
110101	  user registration failed
110161    exceeds the maximum withdrawal support accuracy
120202	  currency is not supported
120206    withdrawal second confirmation failed
120402	  insufficient balance in withdrawal or transfer
120403	  insufficient balance of withdrawal fee
120404	  withdrawal or transfer amount is too small, less than or equal to the commission fee
900006    the user is at risk. Withdrawal is prohibited
3040006   cannot transfer money to itself

========  ==================================================================
