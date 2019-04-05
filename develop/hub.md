# Hub

## Configuration

## API Documentation

Channels

Threads

* [Get Channel](#get-channel)

### Get Channel

  TODO -- fix, this is sample

* **URL:** /users/:id

* **Method:** `GET`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ id : 12, name : "Michael Bloom" }` |
| 404 | NOT FOUND | `{ error : "User doesn't exist" }` |
| 401 | UNAUTHORIZED | `{ error : "You are unauthorized to make this request." }` |

* **Example:**

  ```javascript
    $.ajax({
      url: "/users/1",
      dataType: "json",
      type : "GET",
      success : function(r) {
        console.log(r);
      }
    });
  ```