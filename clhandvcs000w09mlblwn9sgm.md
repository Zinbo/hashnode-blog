---
title: "Mockito Cheatsheet"
datePublished: Mon May 04 2020 15:53:25 GMT+0000 (Coordinated Universal Time)
cuid: clhandvcs000w09mlblwn9sgm
slug: mockito-cheatsheet
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683214541713/5cd2b270-6954-49eb-a4e8-d9c268d3a82e.png
tags: java, testing, mockito

---

Mockito is a great framework for testing in java. I use it all the time and have done for many years now. It works hand in hand with dependnecy injection, a topic I covered in my last blog ["Spring for Humans"](https://stacktobasics.com/spring-for-humans). However I sometimes find it's a victim of it's own success - there's a lot you can do with it so it's easy to forget how to do things!

So here's a cheat sheet which covers most of the features I use in Mockito.

# Setting Up Mockito

## Using JUnit &lt;5 Annotations

```java
import org.mockito.junit.MockitoJUnitRunner;
import org.junit.runner.RunWith;

@RunWith(MockJUnitRunner.class)
public class MyClassTest {
    ...
}
```

## Using JUnit 5 Annotations

```java
import org.mockito.junit.jupiter.MockitoExtension;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
public class MyClassTest {
    ...
}
```

## Using initMocks

Useful if you need more than one RunWith/Extend (e.g. `SpringJunit4ClassRunner`)

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;

public class MyClassTest {
    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
    }
    ...
}
```

# Creating Mocks and Spies

An easy way I use to remember the difference between mocks and spies is:

* Mock: By default, all methods are stubbed unless you say otherwise.
    
* Spy: By default, all methods use real implementation unless you say otherwise.
    

## Creating a Mock

### Use Annotations on Fields

There seems to be a consensus that this is the cleanest way to mock an object (which goes against my brain's aversion for field injection, but that's a story for another day).

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Mock;

public class MyClassTest {

    @Mock
    private MyClass myObject;

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        // myObject is now a mock
    }
    ...
}
```

### Create from Method

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Mock;

public class MyClassTest {

    private MyClass myObject;

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        myObject = Mockito.mock(MyClass.class);
    }
    ...
}
```

### Create from Method using Initialised Object

Useful if you want a mock to be created from an object which has been constructed using a constructor.

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Mock;

public class MyClassTest {
    
    private Person person = Person("name", 18);

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        person = Mockito.mock(person);
    }
    ...
}
```

## Creating a Spy

This is very similar to mocks.

### Use Annotations on Fields

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Spy;

public class MyClassTest {

    @Spy
    private MyClass myObject;

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        // myObject is now a spy
    }
    ...
}
```

### Create from Method

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Spy;

public class MyClassTest {

    private MyClass myObject;

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        myObject = Mockito.spy(MyClass.class);
    }
    ...
}
```

### Create from method using Initialised Object

Useful if you want a spy to be created from an object which has been constructed using a constructor.

```java
import org.mockito.MockitoAnnotations;
import org.junit.Before;
import org.mockito.Spy;

public class MyClassTest {
    
    private Person person = Person("name", 18);

    @Before
    public void before() throw Exception {
        Mockito.initMocks(this);
        person = Mockito.spy(person);
    }
    ...
}
```

# Injecting Mocks

## Using @InjectMocks Annotation

```java
public class InjectTest {

    @Mock
    private BlogRepository blogRepository;
    @InjectMocks
    private BlogPostService blogPostService;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        System.out.println(blogPostService);
    }
}
```

## Using Constructor

```java
public class InjectTest {

    @Mock
    private BlogRepository blogRepository;
    @InjectMocks
    private BlogPostService blogPostService;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        blogPostService = new BlogPostService(blogRepository);
    }
}
```

# Stubbing a method

## Returning from a stubbed Method

```java
@Test
public void getAllBlogPosts_withBlogPostsInDb_returnsBlogPosts() {
    // arrange
    List<BlogPost> expected = Arrays.asList(new BlogPost("Spring for humans"), 
            new BlogPost("Mockito Cheatsheet"));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getAllBlogPosts()).thenReturn(expected);
    BlogPostService service = new BlogPostService(blogRepository);
    
    // act
    List<BlogPost> actual = service.getAllBlogPosts();
    
    // assert
    assertEquals(expected, actual);
}
```

## Providing an Alternative Implementation for a Method

```java
@Test
public void getBlogPostById_withExistingBlogPostInDb_returnsBlogPost() {
    // arrange
    BlogPost expected = new BlogPost("Spring for Humans");
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getBlogPostById(Mockito.anyInt())).thenAnswer(invocationOnMock -> {
        int id = invocationOnMock.getArgument(0);
        if(id == 1) return Optional.of(expected);
        else return Optional.empty();
    });
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    BlogPost actual = service.getBlogPostById(1).get();

    // assert
    assertEquals(expected, actual);
}
```

## Throwing an Exception from a Method

```java
@Test(expected = IllegalArgumentException.class)
public void getBlogPostById_withMissingBlogPost_throwsException() {
    // arrange
    BlogPost expected = new BlogPost("Spring for Humans");
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getBlogPostById(1)).thenThrow(new IllegalArgumentException());
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    BlogPost actual = service.getBlogPostById(1).get();

    // assert
    // Test will pass if exception of type IllegalArgumentException is thrown
}
```

## Calling the Real Implementation

```java
@Test
public void getBlogPostById_withMissingBlogPost_returnsEmpty() {
    // arrange
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getBlogPostById(1)).thenCallRealMethod();
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    Optional<BlogPost> actual = service.getBlogPostById(1);

    // assert
    assertFalse(actual.isPresent());
}
```

## Using Deep Stubs

Deep stubs are used for methods which are chained, but you don't care about the intermediate values. These should be used sparingly, as discussed in the [Mockito documentation](https://www.javadoc.io/static/org.mockito/mockito-core/3.3.3/org/mockito/Mockito.html#RETURNS_DEEP_STUBS).

```java
// Without deep stubs
@Test
public void checkIfDriverIsPostgres_withPostgresDriver_returnsTrue() {
    // arrange
    Configuration configMock = Mockito.mock(Configuration.class);
    RepositoryConfiguration repositoryConfigMock = Mockito.mock(RepositoryConfiguration.class);
    Mockito.when(configMock.getRepositoryConfiguration()).thenReturn(repositoryConfigMock);
    DatabaseConfiguration databaseConfigMock = Mockito.mock(DatabaseConfiguration.class);
    Mockito.when(repositoryConfigMock.getDatabaseConfiguration()).thenReturn(databaseConfigMock);
    DatabaseConfiguration.DriverConfiguration driverConfigMock = Mockito.mock(DatabaseConfiguration.DriverConfiguration.class);
    Mockito.when(databaseConfigMock.getDriverConfiguration()).thenReturn(driverConfigMock);
    Mockito.when(driverConfigMock.getDriverType()).thenReturn("postgres");

    BlogRepository blogRepository = new BlogRepository(configMock);

    // act
    boolean actual = blogRepository.checkIfDriverIsPostgres();

    // assert
    Assert.assertTrue(actual);
}

// With deep stubs
@Test
public void usingDeepStubs_checkIfDriverIsPostgres_withPostgresDriver_returnsTrue() {
    // arrange
    Configuration configMock = Mockito.mock(Configuration.class, Mockito.RETURNS_DEEP_STUBS);
    Mockito.when(configMock.getRepositoryConfiguration()
            .getDatabaseConfiguration().getDriverConfiguration().getDriverType()).thenReturn("postgres");

    BlogRepository blogRepository = new BlogRepository(configMock);

    // act
    boolean actual = blogRepository.checkIfDriverIsPostgres();

    // assert
    Assert.assertTrue(actual);
}
```

# Stubbing a Void Method

## Providing an Alternative Implementation for a Method

```java
@Test
public void getAllBlogPosts_withBlogPostsInDb_callsVerifyConnectionToDatabaseIsAlive() {
    // arrange
    Configuration configMock = Mockito.mock(Configuration.class);
    BlogRepository blogRepository = new BlogRepository(configMock);

    AtomicBoolean verifyMethodCalled = new AtomicBoolean(false);
    Mockito.doAnswer(invocationOnMock -> {
        // We can do whatever we want here, and it will be executed when
        // verifyConnectionToDatabaseIsAlive
        // If the method had any arguments, we can capture them here
        verifyMethodCalled.set(true);
        return null;
    }).when(configMock).verifyConnectionToDatabaseIsAlive();

    // act
    blogRepository.getAllBlogPosts();

    // assert
    assertTrue(verifyMethodCalled.get());
}
```

## Throwing an Exception from a Method

```java
@Test(expected = DatabaseDownException.class)
public void getAllBlogPosts_withConnectionToDbDown_throwsException() {
    // arrange
    Configuration configMock = Mockito.mock(Configuration.class);
    BlogRepository blogRepository = new BlogRepository(configMock);

    AtomicBoolean verifyMethodCalled = new AtomicBoolean(false);
    Mockito.doThrow(new DatabaseDownException()).when(configMock).verifyConnectionToDatabaseIsAlive();

    // act
    blogRepository.getAllBlogPosts();

    // assert
    // Test will pass if exception of type DatabaseDownException is thrown
}
```

## Calling the Real Implementation

```java
@Test
public void getAllBlogPosts_withBlogPostsInDb_returnsBlogPosts() {
    // arrange
    Configuration configMock = Mockito.mock(Configuration.class);
    BlogRepository blogRepository = new BlogRepository(configMock);

    AtomicBoolean verifyMethodCalled = new AtomicBoolean(false);
    Mockito.doCallRealMethod().when(configMock).verifyConnectionToDatabaseIsAlive();

    // act
    blogRepository.getAllBlogPosts();

    // assert
    // Test will pass if exception of type DatabaseDownException is thrown
}
```

# Matchers

## Using a Real Parameter

```java
@Test
public void getBlogPostByAuthorAndAfterDate_withExistingBlogPostsInDb_returnsBlogPosts() {
    // arrange
    List<BlogPost> expected = Collections.singletonList(new BlogPost("Spring for Humans"));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Date date = new Date();
    String author = "Shane";
    Mockito.when(blogRepository.getBlogPostsByAuthorAndAfterDate(author, date)).thenReturn(expected);
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    List<BlogPost> actual = service.getBlogPostsByAuthorAndAfterDate(author, date);

    // assert
    assertEquals(expected, actual);
}
```

## Using any()

```java
@Test
public void getBlogPostByAuthorAndAfterDate_withoutMatchingBlogPostsInDb_returnsEmptyList() {
    // arrange
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getBlogPostsByAuthorAndAfterDate(Mockito.any(), Mockito.any())).thenReturn(Collections.emptyList());
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    List<BlogPost> actual = service.getBlogPostsByAuthorAndAfterDate("any author", new Date());

    // assert
    assertTrue(actual.isEmpty());
}
```

## Using anyClass()

```java
@Test
public void getBlogPostByAuthorAndAfterDate_withMatchingAuthorAndDate_returnsPostsSortedByDate() {
    // arrange
    BlogPost first = new BlogPost("Spring for Humans", "Shane", new Date(5));
    BlogPost second = new BlogPost("Mockito Cheatsheet", "Shane", new Date(6));
    BlogPost third = new BlogPost("?", "Shane", new Date(7));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    // Good for overloaded methods, you can specify type of params so the call isn't ambiguous.
    Mockito.when(blogRepository.getBlogPostsByAuthorAndAfterDate(Mockito.any(String.class), Mockito.any(Date.class)))
            .thenReturn(Arrays.asList(second, third, first));
    BlogPostService service = new BlogPostService(blogRepository);

    List<BlogPost> expected = Arrays.asList(first, second, third);

    // act
    List<BlogPost> actual = service.getBlogPostsByAuthorAndAfterDate("any author", new Date());

    // assert
    assertEquals(expected.get(0), actual.get(0));
    assertEquals(expected.get(1), actual.get(1));
    assertEquals(expected.get(2), actual.get(2));
}
```

## Using eq()

```java
@Test
public void getBlogPostByAuthorAndAfterDate_withMatchingAuthorButFutureDate_returnsEmptyList() {
    // arrange
    List<BlogPost> expected = Collections.singletonList(new BlogPost("Spring for Humans"));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Date date = new Date();
    Mockito.when(blogRepository.getBlogPostsByAuthorAndAfterDate(Mockito.any(), Mockito.eq(date))).thenReturn(expected);
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    List<BlogPost> actual = service.getBlogPostsByAuthorAndAfterDate("any author", date);

    // assert
    assertTrue(actual.isEmpty());
}
```

## Using Custom Matcher

```java
@Test
public void checkIfBlogPostHasBeenSaved_withBlogPost_returnsTrue() {
    // arrange
    BlogPost post = new BlogPost("Spring for Humans", "Shane", new Date(5));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);

    ArgumentMatcher<BlogPost> blogPostMatcher = passedBlogPost ->
            "Shane".equals(passedBlogPost.getAuthor());

    Mockito.when(blogRepository.checkIfBlogPostHasBeenSaved(Mockito.argThat(blogPostMatcher)))
            .thenReturn(true);
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    boolean actual = service.checkIfBlogPostHasBeenSaved(post);

    // assert
    assertTrue(actual);
}
```

# Verify a Stubbed Method Has Been Called

```java
@Test
public void getAllBlogPosts_withBlogPost_verifiesThatRepositoryWasCalled() {
    // arrange
    List<BlogPost> expected = Arrays.asList(new BlogPost("Spring for humans"),
            new BlogPost("Mockito Cheatsheet"));
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    Mockito.when(blogRepository.getAllBlogPosts()).thenReturn(expected);
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    service.getAllBlogPosts();

    // assert
    Mockito.verify(blogRepository).getAllBlogPosts();
}
```

# Verify a Stubbed Method Hasn't Been Called

```java
@Test
public void getBlogPostById_withBlogPost_verifiesThatCorrectMethodWasCalled() {
    // arrange
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    BlogPost expoected = new BlogPost("Spring for humans");
    Mockito.when(blogRepository.getBlogPostById(1)).thenReturn(Optional.of(expoected));
    BlogPostService service = new BlogPostService(blogRepository);

    // act
    service.getBlogPostById(1);

    // assert
    Mockito.verify(blogRepository, Mockito.never()).getBlogPostById(100);
    Mockito.verify(blogRepository).getBlogPostById(1);
}
```

## Capturing Arguments

```java
@Test
public void getBlogPostById_withBlogPost_verifiesCorrectArgumentPassed() {
    // arrange
    BlogRepository blogRepository = Mockito.mock(BlogRepository.class);
    int expected = 1;
    Mockito.when(blogRepository.getBlogPostById(expected))
            .thenReturn(Optional.of(new BlogPost("Spring for humans")));
    BlogPostService service = new BlogPostService(blogRepository);
    ArgumentCaptor<Integer> captor = ArgumentCaptor.forClass(Integer.class);

    // act
    service.getBlogPostById(expected);
    Mockito.verify(blogRepository).getBlogPostById(captor.capture());
    int actual = captor.getValue();

    // assert
    assertEquals(expected, actual);
}
```

# Conclusion

Hopefully the snippets laid out here will help you quickly set up your mockito tests. The full examples can be seen [here on GitHub](https://github.com/Zinbo/blog-examples).

Happy Coding!