# Alternate key

Microsoft Graph API Design Pattern

*The alternate key pattern provides the ability to query for a single, specific resource identifiable via one of an alternative set of properties that is not its primary key.*

## Problem

The resources exposed in Microsoft Graph are identified through a primary key, which guarantees uniqueness inside the same resource collection. Often though, that same resource can also be uniquely identified by an alternative, more convenient property that provides a better developer experience.

Take a look at the `user` resource: while the `id` is the typical way to get the resource details, the `mail` address is also a unique property that can be used to identify it.

The resource can be accessed using the `$filter` query parameter, such as

```http
GET https://graph.microsoft.com/v1.0/users?$filter=mail eq 'bob@contoso.com'
```
However, in this case, the returned result is wrapped in an array that needs to be unpacked. When the uniqueness of the property within the collection implies that only zero or one results can be returned from the call this array provides a suboptimal experience for callers.

## Solution

Typically resources in Graph are accessed using a simple forward-slash delimited URL pattern (this pattern is sometimes referred to as key-as-segment).

```http
https://graph.microsoft.com/v1.0/users/0 - Retrieves the employee with ID = 0.
```

However, resources can also be accessed using parentheses to delimit the key, like this:

```http
https://graph.microsoft.com/v1.0/users(0) - Also retrieves the employee with ID = 0.
```

Resource addressing by using an alternative key can be achieved by using this same parentheses-style convention with one difference: alternate keys MUST specify the key property name to unambiguously determine the alternate key, like this:

```http
https://graph.microsoft.com/v1.0/users(email='bob@contoso.com') Retrieves the employee with the email matching `bob@contoso.com`.
```

In the same way as requesting a resource via the canonical key, if a resource cannot be located that matches the alternate key, then a 404 must be returned.

> **Note:** When requesting a resource via alternate keys, the simple slash-delimited URL style does not work.

> **Note:** Do not use multi-part alternate keys.   Feedback has been that customers find multi-part keys confusing.
> Either create a composite single-part surrogate key property or fall back to logical operations in a $filter clause.

## When to use this pattern

Use this pattern when your resource type has other keys than its canonical key which uniquely identify a single resource.

## Example

The same user is identified via the alternate key SSN, the canonical (primary) key ID using the non-canonical long form with a specified key property name, and the canonical short form without a key property name.

Declare `mail` and `ssn` as alternate keys on an entity:

```xml
<EntityType Name="user">
   <Key>
     <PropertyRef Name="id" />
   </Key>
   <Property Name="id" Type="Edm.Int32" />

   <Property Name="mail" Type="Edm.String" />
   <Property Name="ssn" Type="Edm.String" />
   <Annotation Term="OData.Community.Keys.V1.AlternateKeys">
      <Collection>
         <Record Type="OData.Community.Keys.V1.AlternateKey">
            <PropertyValue Property="Key">
               <Collection>
                  <Record Type="OData.Community.Keys.V1.PropertyRef">
                     <PropertyValue Property="Name" PropertyPath="mail" />
                  </Record>
               </Collection>
            </PropertyValue>
         </Record>
         <Record Type="OData.Community.Keys.V1.AlternateKey">
            <PropertyValue Property="Key">
               <Collection>
                  <Record Type="OData.Community.Keys.V1.PropertyRef">
                     <PropertyValue Property="Name" PropertyPath="ssn" />
                  </Record>
               </Collection>
            </PropertyValue>
         </Record>
      </Collection>
   </Annotation>
</EntityType>
```

1. Get a specific resource through `$filter`:

    ```http
    GET https://graph.microsoft.com/v1.0/users/?$filter=ssn eq '123-45-6789'
    ```
    
    ```json
    {
      "value": [
        {
          "givenName": "Bob",
          "jobTitle": "Retail Manager",
          "mail": "bob@contoso.com",
          "mobilePhone": "+1 425 555 0109",
          "officeLocation": "18/2111",
          "preferredLanguage": "en-US",
          "ssn": "123-45-6789",
          "surname": "Vance",
          "userPrincipalName": "bob@contoso.com",
          "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66"
        }
      ]
    }
    ```

2. Get a specific resource either through its primary key or through the two alternate keys:

    ```http
    GET https://graph.microsoft.com/v1.0/users/1a89ade6-9f59-4fea-a139-23f84e3aef66
    GET https://graph.microsoft.com/v1.0/users(1a89ade6-9f59-4fea-a139-23f84e3aef66)
    GET https://graph.microsoft.com/v1.0/users(ssn='123-45-6789')
    GET https://graph.microsoft.com/v1.0/users(mail='bob@contoso.com')
    ```
   
    All four yield the same response:
    
    ```json
    {
      "givenName": "Bob",
      "jobTitle": "Retail Manager",
      "mail": "bob@contoso.com",
      "mobilePhone": "+1 425 555 0109",
      "officeLocation": "18/2111",
      "preferredLanguage": "en-US",
      "ssn": "123-45-6789",
      "surname": "Vance",
      "userPrincipalName": "bob@contoso.com",
      "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66"
    }
    ```

3. Request a resource for an unsupported alternate key property:

    ```http
    GET https://graph.microsoft.com/v1.0/users(name='Bob')
    
    400 Bad Request
    {
        "error" : {
            "code" : "400",
            "message": "'name' is not a valid alternate key for the resource type 'user'."
        }
    }
    ```

4. Request a resource where the alternate key property does not exist on any resource in the collection:

    ```http
    GET https://graph.microsoft.com/v1.0/users(email='unknown@contoso.com')
    
    404 Not Found
    {
        "error" : {
            "code" : "404",
            "message": "No user with the the specified 'email' could be found."
        }
    }
    ```
