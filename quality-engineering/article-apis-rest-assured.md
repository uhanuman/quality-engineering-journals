
## How to log response in Rest Assured only in the case of an error.
User ifError() method. It Logs everything only if an error occurs.
```java
    public void logOnlyIfError()
    {
    
    RestAssured.given().baseUri("http://localhost:4001")
    .contentType(ContentType.JSON).log().all().when().get("/users").
                    then().log().ifError().assertThat().statusCode(200);
    }

```

##  How to mask header information in API testing?

```java

    Set headers= new HashSet();
        headers.add("x-api-key");
        headers.add("secreat key");
    RestAssured.given().config(config().logConfig(logConfig()
    .blacklistHeaders(headers)))
```

## What is the difference between path params and query params and how to use them?
```java
    @Test
    void test() {
        RestAssured.given()
        .pathParam("user", "users/qaa   utomatonhub")
        .when()
        .get("https://http://localhost:4001/{user}")
        .prettyPrint();
    }
    public void verifyRepos(){
        
      RestAssured.given()
          .queryParam("sort","created")
          .queryParam("direction","desc")
          .when().get(""utf-8">https://http://localhost:4001/users")
          .then().log().body();
    }


```

## How to use Basic Authentication in Rest Assured?
Basic authentication is used to ensure that only the authorized users can access/View the API’s.
```java
RestAssured.given().auth().basic(username, password)
   or
RestAssured.given().auth().preemptive().basic("username", "password")
```


## How to download a file using rest assured?
```java
public void test() {
    byte[] dowloadedFile = RestAssured.given()
    .when().get("https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&thumbs=True")		 
    .then().extract().asByteArray();
    System.out.println("Download File Size : "+dowloadedFile.length);
}
```

## How to upload a media file using Rest Assured?
```java
public void upload() {
	File file = new File("myavatar.png");
	Response resp= RestAssured.given()
	.multiPart("media_url", file,"application/octet-stream").when()
        .post("https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&thumbs=True")
       .then().extract().response();
	String url=resp.path("url");
	System.out.println(url);
}
```

##   How to perform API chaining(Pass value from one API to another API)?
```java

@Test
public void createBooking(ITestContext context)
{
	int bookingId = RestAssured
	.given()
		.log().all()
        .timeout(5000).
		.baseUri("https://restful-booker.herokuapp.com/")
		.basePath("booking")
		.contentType(ContentType.JSON)
		.body("booking.json")
	.when()
		.post()
	.then()
		.log().all()
		.extract().jsonPath().get("bookingid");
		// Storing data in a context to use for other tests
		context.setAttribute("bookingId", bookingId);
	}

    @Test
    public void updateBooking(ITestContext context)
    {
    int bookingId = (int) context.getAttribute("bookingId");
    RestAssured.given().log().all()
    .baseUri("https://restful-booker.herokuapp.com/")
    .basePath("booking/"+bookingId)
    .header("Authorization","Your token").contentType(ContentType.JSON)
    .body("booking.json").when().put().then().log().all();
    }
```

## Why does Rest Assured use static import?
Static import is a feature of the Java programming language that allows you to use members such as fields and methods that were scoped as public static in their container class without naming the class in which they were defined.
```java

```

## How to check that a specific item is present in a collection using Rest Assured?
In Rest Assured, hasItem() function checks if a specified item exists in a collection. Example: .body("items", hasItem("value")).It is part of the Hamcrest Matcher Library.
```java
        RestAssured
            .when().get(endpoint)
            .then()
            .statusCode(200)  // Assuming the response code is 200
            .body("items", Matchers.hasItem("text1"))  
            }
```

## 
```java

```
## 
```java

```


## 
```java

```

## 
```java

```

## 
```java

```
## 
```java

```