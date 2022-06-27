# URL Shortner Service
Designed and built a URL shortener service to shorten billions of URLs - assume this will support a company that provides tracking of packages to a few hundred top retailers.
The service returns a short URL, given a longer URL; and similarly redirects to the longer URL when given the short URL. As part of your solution, provide a brief technical document that outlines some of your design decisions such as storage choices, choice of language, length of the short URL, rate limits etc. Extra points for providing your solution in docker / kubernetes setup.

Please have a look at the details and feel free to write/ call back in case of any queries?

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

Our System should be able to generate a unique url which is shorten form of the input url.

Lets say, we want to shorten the url "https://corp.nagarro.com/" we would get some output like "https://shorturl/pQG12a".
In the above example "https://shorturl/" will always fixed and is called the base URL. So our main work here will be to generate the part "pQG12a"from the           input url i.e "https://corp.nagarro.com/".
If we add both the base and the required String we will get the shorten URL. Lets give a template to the output as "base_url/<hash_value>"

We may have to shorten hundreds of millions of URLs lets say a billion of url. So we should be able to generate billions of the unique <hash_value>. If we use
62 characters(uppercase and lowercase alphabets and digits) to generate all possible <hash_value> of length 5, we would be able to generate 62^5 i.e 0.9 billion <hash_value> which is quite close to our expectation. 

Lets say this <hash_value> is always of fixed length i.e 5. We can randomly generate this <hash_value> and store it in the form of <ID,URL> in a list. Where ID will be the randomly generated <hash_value>. So when the user input the URL we generate the <hash_value> as the ID and store it in the list. While when we are typing the shorten_url in our browser we can search for the <hash_value> in the list and redirect to the URL.

A problem in the approach is we might generate same <hash_value> for different URL.
To overcome this hurdle we may store it in the database in the following way:-

```
Table Short_Url(ID : int PRIMARY_KEY AUTO_INC,Original_url : varchar,Short_url : varchar)
```
In this the ID will always be unique unique. We can use this ID to generate a Short_url which is always unique.
One such straight forward approach to generate unique Short_url from the ID is this:-
(Java Code)
```
static String idToShortURL(long n){
  // Map to store 62 characters
  String varValue = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  char map[] = varValue.toCharArray();

  StringBuilder shorturl = new StringBuilder();
  // Convert given integer id to a base 62 number
  while (n != 0){
      shorturl.append(map[(int) (n%62)]);
      n = n/62;
  }
  // Reverse shortURL to complete base conversion
  return shorturl.toString();
}
```

To convert Short_url to id:-
(Java Code)
```
static long shortURLtoID(String shortURL){
    long id = 0;
    int ind = shortURL.length();
    char[] map = new char[ind];
    for(char c : shortURL.toCharArray()) map[--ind] = c;
  
    // A simple base conversion logic
    for (;ind < map.length; ind++){
        if ('a' <= map[ind] && map[ind] <= 'z')
          id = id*62 + map[ind] - 'a';
        if ('A' <= map[ind] && map[ind] <= 'Z')
          id = id*62 + map[ind] - 'A' + 26;
        if ('0' <= map[ind] && map[ind] <= '9')
          id = id*62 + map[ind] - '0' + 52;
    }
    return id;
}

```
