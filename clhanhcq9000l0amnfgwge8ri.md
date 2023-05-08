---
title: "Using Jackson Subtypes to Write Better Code"
datePublished: Fri Apr 29 2022 15:54:36 GMT+0000 (Coordinated Universal Time)
cuid: clhanhcq9000l0amnfgwge8ri
slug: jackson-sub-types
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683216109630/c3601d17-97ce-47f6-bf99-6b1c654ff4b8.png
tags: java, json, rest-api, jackson

---

In this post, we'll look at how Jackson Subtypes can automatically map JSON to the right subclass, without having to write your own converter.  
First, we'll give some background into why this is useful, and then we'll go through some practical examples for two different use cases.

# Why are Jackson Subtypes Useful?

You'll sometimes find that you receive JSON HTTP responses that look similar but have a few fields which are different. If you have to map these to Java classes then it can be annoying to have to replicate all of the common fields across each of your DTO classes.

Let's look at an example of the problem.

Let's say you get a response from a service that looks like this:

```json
[
  {
    "id": "1",
    "type": "onlineOrder",
    "quantity": 1,
    "order": {
      "name": "Playstation 5",
      "onlineCode": "123456",
      "referralId": "6789",
      "customerId": "5"
    }
  },
  {
    "id": "2",
    "type": "shopOrder",
    "quantity": 3,
    "order": {
      "name": "Xbox One",
      "shopId": "2",
      "paymentType": "card"
    }
  }
]
```

We can see that the two objects in the list look similar, but both have a different `order` object format.  
We could map these to DTO objects using the following class:

```java
public class GenericOrderDTO {
    private String id;
    private String type;
    private int quantity;
    private Object order;
    
    // getters and setters omitted
}
```

This will work, but it won't give us any indication about what is inside the order object, plus we'll have to do some casting.  
We could also use `Map<String, String>` instead to represent the order object, which will give us a map of field name to value.  
In either case, we would need to write our own mapper to figure out if the order is a shop type or an online type and map the fields accordingly.

A better way to do this is to use Jackson's `@JsonTypeInfo` and `@JsonSubTypes` annotations with inheritance.

# How Jackson Subtypes Work

Jackson has a concept of subtypes, which is similar to a subclass.

With inheritance you have your parent class which contains common fields across each of your subclasses, and your subclasses which contain fields unique to them. Subtypes work in a similar way, with the help of some annotations.

In order to use subtypes we need to have some property in the JSON that tells us which subtype to use. In our example above we have the `type` field, which states whether the `order` object is an online order or a shop order. Another way is when the type field is inside the subtype itself.

Let's look at both.

## Type Field is on the Root Object

Using our example above, we can map the JSON to the following DTOs:

```java
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class OrderDTO {
    private String id;
    private int quantity;
    private String type;
    @JsonTypeInfo(
            use = JsonTypeInfo.Id.NAME,
            include = JsonTypeInfo.As.EXTERNAL_PROPERTY,
            property = "type",
            defaultImpl = UnknownTypeOrderDTO.class
    )
    private OrderTypeDTO orderTypeDTO;

    // getters and setters omitted
}
```

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;

@JsonSubTypes({
        @JsonSubTypes.Type(value = OnlineTypeOrderDTO.class, name = "onlineOrder"),
        @JsonSubTypes.Type(value = ShopTypeOrderDTO.class, name = "shopOrder")
})
public class OrderTypeDTO {
}
```

```java
public class OnlineTypeOrderDTO extends OrderTypeDTO {
    private String name;
    private String referralId;
    private String customerId;

    // getters and setters omitted
}
```

```java
public class ShopTypeOrderDTO extends OrderTypeDTO {
    private String name;
    private String shopId;
    private String paymentType;

    // getters and setters omitted
}
```

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class UnknownTypeOrderDTO extends OrderTypeDTO {

}
```

Looking at `OrderTypeDTO` first, we have added an annotation that lists what class should map to what name.

In the `OrderDTO` class, we include an annotation that tells Jackson to map the `orderTypeDTO` to the class where the value of the `type` field matches the name specified in the `@JsonSubTypes` annotation.  
We also state that the `type` field is an external property. That is, it's not a field on `OrderTypeDTO`, but rather on `OrderDTO`.  
Finally, we state that if Jackson can't find a matching class based on the value of `type`, then use `UnknownTypeOrderDTO` instead. This is protect us from exceptions where the order endpoint might add a new type before we change our code. You can then have some code that appropriately deals with any orders that map to `UnknownTypeOrderDTO`.

## Type field is on the Subtype Object

Another way is when the type field is on the subtype object itself.  
Let's change our example slightly such that the JSON we receive looks like this:

```json
[
  {
    "id": "1",
    "quantity": 1,
    "order": {
      "name": "Playstation 5",
      "type": "onlineOrder",
      "onlineCode": "123456",
      "referralId": "6789",
      "customerId": "5"
    }
  },
  {
    "id": "2",
    "quantity": 3,
    "order": {
      "name": "Xbox One",
      "type": "shopOrder",
      "shopId": "2",
      "paymentType": "card"
    }
  }
]
```

In this case, our DTOs would look like this:

```java
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class OrderDTO {
    private String id;
    private int quantity;
    @JsonTypeInfo(
            use = JsonTypeInfo.Id.NAME,
            property = "type",
            defaultImpl = UnknownTypeOrderDTO.class
    )
    private OrderTypeDTO orderTypeDTO;

    // getters and setters omitted
}
```

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;

@JsonSubTypes({
        @JsonSubTypes.Type(value = OnlineTypeOrderDTO.class, name = "onlineOrder"),
        @JsonSubTypes.Type(value = ShopTypeOrderDTO.class, name = "shopOrder")
})
public class OrderTypeDTO {
    private String type;
}
```

The `ShopTypeOrderDTO`, `OnlineTypeOrderDTO`, and `UnknownTypeOrderDTO` classes are exactly the same as before.

The difference here is that the `type` field has moved to the `OrderTypeDTO` class, and `include =` [`JsonTypeInfo.As`](http://JsonTypeInfo.As)`.EXTERNAL_PROPERTY` was removed from the `@JsonTypeInfo` annotation.

# Conclusion

In this post, we've expressed when and why Subtypes are useful, as well as learned how to use them when you have the type inside or outside of the subclass through practical examples.

Till next time!