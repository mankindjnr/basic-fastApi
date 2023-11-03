# FastAPI

### installation
pip install fastapi
pip install uvicorn

uvicorn -> ASGI server, what it does is it takes the request and sends it to the FastAPI application and then FastAPI application will process it and send it back to the client.

To ensure that fastapi is installed, run the following command in the terminal
```bash
pip show fastapi
```
or
inside the working folder, create a file and import fastapi then run that file

```working.py```
```python
import fastapi
```

## Writing the first API
```python
from fastapi import FastAPI

app = FastAPI()
```
After the above set up, you can go on and write your endpoints
```python
@app.get("/")
def home():
    return {"message": "Hello World"}
```
Now when we run the endpoint, we get the following response
```json
{
    "message": "Hello World"
}
```

## Running the server
cd to the folder where your files are located and run the following command
```bash
uvicorn working:app --reload
```
where __working__ is the name of the file and __app__ is the name of the FastAPI instance create at the top of the file

With that simple endpoint, __FastAPI__ automatically generates a __swagger__ documentation for us. To access the documentation, go to the base url ```/docs```

## Path Parameters
```python
inventory = {
    1: {
        "name": "milk",
        "price": 3.99,
        "brand": "Clover"
    },
    2: {
        "name": "bread",
        "price": 2.99,
        "brand": "Wonder"
    }
}

@app.get("/get-item/{item_id}")
def get_item(item_id: int):
    return inventory[item_id]

@app.get("/get-by-nameandid/{item_id}/{name}")
def get_item(item_id: int, name: str):
    return inventory[item_id]
```

The snippet above simulates a database. The ```get_item``` function takes in an ```item_id``` and returns the item with that id. The other ```get_item``` function takes in an ```item_id``` and a ```name``` and returns the item with that id and name.

You can also add descriptions to your endpoints
```python
@app.get("/get-item/{item_id}")
def get_item(item_id: int = Path(None, description="The ID of the item you would like to view", gt=0)):
    return inventory[item_id]
```
The snippet above adds a description to the ```item_id``` parameter. The ```gt``` parameter ensures that the ```item_id``` is greater than 0. The __None__ parameter is the default value of the ```item_id``` parameter. (currently the path cannot have a default value, so none is not needed)
__Description__ is displayed in the swagger documentation to help the user understand what the parameter is for.

## Query Parameters
They come after the question mark in a url in form of ```key=value```. They are used to filter data. They are optional and can have default values.

***
### An example of required query parameters

```python
@app.get("/get-by-name")
def get_item(name: str):
    for item_id in inventory:
        if inventory[item_id]["name"] == name:
            return inventory[item_id]
    return {"Data": "Not found"}
```

The snippet above takes in a ```name``` parameter and returns the item with that name. If the item is not found, it returns a message saying that the item was not found.

```127.0.0.1:8000/get-by-name?name=milk``` will return the following response
```json
{
    "name": "milk",
    "price": 3.99,
    "brand": "Clover"
}
```
if the url is run without any paramters, then there will be an error.
To ensure there are no errors, you can make the query parameters optional:

***
### An example of optional query parameters
To make the parameter optional, you can add a default value to it
```python
from typing import Optional

@app.get("/get-by-name")
def get_item(name: Optional[str] = None):
    if name:
        for item_id in inventory:
            if inventory[item_id]["name"] == name:
                return inventory[item_id]
        return {"Data": "Not found"}
    return inventory
```

To add more query parameters:
```python
@app.get("/get-by-name")
def get_item(name: Optional[str] = None, test: int):
    if name:
        for item_id in inventory:
            if inventory[item_id]["name"] == name:
                return inventory[item_id]
        return {"Data": "Not found"}
    return inventory
```
We have added mandatory query parameter __test__, when a mandatory query is added after an aoptional query, python throws an error which can be fixed it two ways:
1: reorder them so the optional query is after the mandatory query
2: add an asterisk ```get_item(*, name: Optional[str] = None, test: int)```

## Combining query parameters and path parameters
```python
@app.get("/get-by-name/{item_id}")
def get_item(item_id: int, name: Optional[str] = None, test: int):
    if name:
        for item_id in inventory:
            if inventory[item_id]["name"] == name:
                return inventory[item_id]
        return {"Data": "Not found"}
    return inventory[item_id]
```

When combining query parameters, ensure that the path parameter is also included in the function parameters, thats why we have ```item_id: int``` in the function parameters. Past that we have the optional query parameter ```name: Optional[str] = None``` and the ```test``` as a mandatory query parameter.

## Request Body
The request body is the data that is sent by the client to the server. It is sent in the form of a json object. To access the request body, you need to create a model that will be used to parse the data from the request body.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    brand: Optional[str] = None
```
This class will define the kind of data we intend to accept from the client. The ```BaseModel``` class is imported from the ```pydantic``` module. The ```BaseModel``` class is used to create a model that will be used to parse the data from the request body.

```python
@app.post("/create-item")
def create_item(item: Item): # this is equal to the class we created above
    return inventory
```

```item: Item``` this tells the function that it is not expecting a path parameter or a query parameter, but a request body. The ```Item``` class is the model that will be used to parse the data from the request body.

In our create_item, we want to include an id for the item being created, for this, we will include a path parameter ```item_id```
```python
@app.post("/create-item/{item_id}")
def create_item(item_id: int, item: Item):
    if item_id in inventory:
        return {"Error": "Item ID already exists"}
    
    inventory[item_id] = {"name": item.name, "price": item.price, "brand": item.brand}
```
So, we first check if the id already exists, if it does, throw an error, if it doesn't, add the item to the inventory.

A better way to insert the data as opposed to ```inventory[item_id] = {"name": item.name, "price": item.price, "brand": item.brand}``` is ```inventory[item_id] = item```
```python
@app.post("/create-item/{item_id}")
def create_item(item_id: int, item: Item):
    if item_id in inventory:
        return {"Error": "Item ID already exists"}
    inventory[item_id] = item
```
And if we are going to insert them this way, then how we search for this elements needs to change as well
```python
@app.get("/get-by-name")
def get_item(name: Optional[str] = None):
    if name:
        for item_id in inventory:
            if inventory[item_id].name == name:
                return inventory[item_id]
        return {"Data": "Not found"}
    return inventory
```