# Consuming and API

For the sake of this example, we are going to start with our CRUD API constructed and running in _localhost:3000_. This will be a message app, so the API will comunitcate with our PostgreSQL database to store all the messages in one table.

The API has 2 different endpoints where we can perform the following actions:
- GET all of the messages using _/messages_.
- POST a new message using _/messages_.
- PUT, to update an existing message using _/messages/:id_, with _id_ being the message's ID from the database.
- DELETE an existing message using _/messages/:id_.
- GET one specific message using _/messages/:id_.

Our message object only has three fields in our database:
- **id**: Unique key to identify one message.
- **title**
- **description**

## Creating the message's model.
The message's model will define the structure of a message object in our frontend and it's very simple. We create a _Models_ folder, and a _message.dto.ts_ file inside it.

```ts
// In message.dto.ts

export class MessageDTO {
    id!: number;
    title: string;
    description: string;
  
    constructor(title: string, description: string) {
      this.title = title;
      this.description = description;
    }
}
```

The only important thing to mention is that we might not provide an ID for every message we invoke the constructor for. This is because, for example, if we are going to create a new message in our database using the _POST /messages_ endpoint, the API automaticly reads the ID of the las message on the table and makes a new one for the new message.

But, for example, if we're going to edit an existing message using the _PUT /messages/:id_ enpoint, we will bring that message from the database to the frontend, which will create a new MessageDTO object which will need to store the ID of the message for when we send it back (updated) to the database.

## Message's service

This service is the way the frontend will use to comunicate with the backend's API.

This communication will be made using the _HttpClient_ module.

So, first, we need to create our service's files using the Angular CLI. So we run `ng generate service Services/message`.

Now, we have a service folder and the basic structure of our service.

1. First, we import the desired Http modules and our MessageDTO data structure.

```ts
// In message.service.ts

import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { MessageDTO } from '../Models/message.dto';
```

2. Now, we need an attribute that saves the API's URL. We don't want anyone accesing to it, so we'll make it private.

```ts
@Injectable({
  providedIn: 'root'
})
export class MessageService {
  private urlMessageApi: string = 'http://localhost:3000/messages';

  constructor(private http: HttpClient) {}  // HttpClient declaration
}
```

Also, so be able to use HttpClient's functions, we need to declare an argument in our constructor.

3. To be able to use the _HttpClient_ we also have to add its provider to the providers array of our app. To do that we need to go to _app.config.ts_

```ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';       // Import this

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes),
    provideHttpClient()     // Add its function to the providers array
  ]
};
```

4. It's time to make the methods that our frontend will use to access the API.

```ts
// In message.service.ts

export class MessageService {
  private urlMessageApi: string = 'http://localhost:3000/messages';

  constructor(private http: HttpClient) {}

  getMessages(): Promise<MessageDTO[]> {    // GET /messages
    return this.http
      .get<MessageDTO[]>(this.urlMessageApi)
      .toPromise() as Promise<MessageDTO[]>;
  }

  getMessageById(msgId: number): Promise<MessageDTO> {  // GET /messages/:id
    return this.http
      .get<MessageDTO>(this.urlMessageApi + '/' + msgId)
      .toPromise() as Promise<MessageDTO>;
  }

  updateMessage(msgId: number, msg: MessageDTO): Promise<MessageDTO> {  // PUT /messages/:id
    return this.http
      .put<MessageDTO>(this.urlMessageApi + '/' + msgId, msg)
      .toPromise() as Promise<MessageDTO>;
  }
}
```

All requests to an API must return a promise of some sort because they are async functions. For example, our _getMessages()_ method returns a _Promise<MessageDTO[]>_ object: a promise of an array of messages, because this method will return all messages of our database.

You might have noticed that we haven't implemented our DELETE method yet. That's because it's a little bit different.

Most DELETE functions usually return information about what they just deleted. In this case, we will implement an example interface that will hold the ID of the deleted message. This interface will be returned by our DELETE method.

```ts
// In message.service.ts

import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { MessageDTO } from '../Models/message.dto';

interface deleteResponse { // This is our new interface
  affected: number;
}

@Injectable({
  providedIn: 'root',
})
export class MessageService {
  // ...

  deleteMessage(msgId: number): Promise<deleteResponse> {
    return this.http
      .delete<deleteResponse>(this.urlMessageApi + '/' + msgId)
      .toPromise() as Promise<deleteResponse>;
  }
}
```

**NOTE**: Requests to APIs can fail and return _undefined_ instead of the expected type of promise. For that we use the `.catch(error => {});` block to handle errors, and once it's done, we can assure Angular that the expected type will be returned. That's why we add `as Promise<...>` at the end of every return statement.

5. We also want to make a method that displays information about errors in th console, and another one to make different pauses during the runtime of the app (for design pourposes).

```ts
// In message.service.ts

errorLog(error: HttpErrorResponse): void {
    console.error('An error occurred:', error.error.msg);
    console.error('Backend returned code:', error.status);
    console.error('Complete message was::', error.message);
}

async wait(ms: number) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
```

## Components

Now, we are going to create our two components and prepare the settings to be able to display them on their apropiate routes.

1. First, we run `ng generate component Components/message-list` and `ng generate component Components/message-form`. This will create the _Components_ folder and one subfolder for every component.

2. Now, we need to import them into our _app.component.ts_ file:

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

import { MessageFormComponent } from './Components/message-form/message-form.component';
import { MessageListComponent } from './Components/message-list/message-list.component';    // IMPORTS

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, MessageFormComponent, MessageListComponent],      // IMPORTS
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'blog-front-own';
}
```

3. Then, we have to specify which routes we want our components to display in:

```ts
// In app.routes.ts

import { Routes } from '@angular/router';
import { MessageListComponent } from './Components/message-list/message-list.component';
import { MessageFormComponent } from './Components/message-form/message-form.component';

export const routes: Routes = [
    {
        path: '',
        component: MessageListComponent
    },
    {
        path: '/:id',
        component: MessageFormComponent
    }
];
```

We are going to display our message list in the root route, and if we specify a valid ID in the URL, it will take us to the message form to be able to create or update any post.

## MessageList component

This component will show a list of all messages, including their IDs, titles and short descriptions. Every message will have its own _Update_ and _Delete_ buttons. We also will implement a _Create Message_ button that will take us to and empty form.

1. First, we will need to import all necesary modules:

```ts
// In message-list.component.ts

import { Component } from '@angular/core';
import { Router } from '@angular/router';       // To redirect to different routes manually
import { CommonModule } from '@angular/common'; // To be able to use the NgFor loop
import { MessageDTO } from '../../Models/message.dto';
import { MessageService, deleteResponse } from '../../Services/message.service';
```

2. Then, we need an attribute to hold all of the messages, and a way to insert them into said attribute.

```ts
// In message-list.component.ts

export class MessageListComponent {
  messages!: MessageDTO[];

  constructor(private messageService: MessageService, private router: Router){
    this.loadMessages();
  };

  private async loadMessages(): Promise<void> {
    try {
      this.messageService.getMessages()
      .then((messageList: MessageDTO[]) => {
        this.messages = messageList;
      });
    } catch (error: any) {
      this.messageService.errorLog(error);
    }
  }
}
```

The attribute _messages_ is a _MessageDTO_ array. Since we are going to use it in the _NgFor_ loop in our HTML, we have to keep it public.

We load the messages using the _getMessages()_ method from the _MessageService_ we implemented before. Since this method returns a _Promise<MessageDTO[]>_, we have to run the returned promise to assign the _MessageDTO[]_ to the _messages_ attribute, since that's its type.

3. Since the form to create and update the messages is in a different route, we have to redirect the user to the proper URL when they click the buttons. That's why we make the following methods inside of our _MessageList_ class.

```ts
createMessage(): void {
    this.router.navigateByUrl('/message/create');
}

updateMessage(messageId: number): void {
  this.router.navigateByUrl('/message/' + messageId);
}
```

4. Finally, we don't want the user to be able to accidentally delete messages, so we use the _confirm()_ function to show a confirmation message which they have to accept. If they do, we run the _deleteMessage()_ method from _MessageService_.

```ts
async deleteMessage(messageId: number): Promise<void> {
    let result = confirm('Do you really want to delete the message with ID: ' +  messageId + '?');

    if (result) {
      try {
        this.messageService.deleteMessage(messageId)
        .then((affectedInterface: deleteResponse) => {
          this.loadMessages();
          alert('Deleted message with ID: ' + affectedInterface.affected);
        });
      } catch(error: any) {
        this.messageService.errorLog(error);
      }
    }
  }
}
```

### HTML for MessageList

For the HTML structure of the list, we are going to make a HTML table, with a generic header, and every other row will be generated by a loop.

```html
<!-- In message-list.component.html -->

<h1>Message list</h1>
<div class="main-container">
    <button (click)="createMessage()">Create message</button>

    <section class="message-list">
        <table>
            <tr>    <!-- Generice header -->
              <th>ID</th>
              <th>Title</th>
              <th>Description</th>
              <th>Actions</th>
            </tr>
            
            <tr *ngFor="let message of messages">   <!-- Content generated by NgFor loop -->
              <td>{{message.id}}</td>
              <td>{{message.title}}</td>
              <td>{{message.description}}</td>
              <td>
                <button (click)="updateMessage(message.id)">Update</button>
                <button (click)="deleteMessage(message.id)">Delete</button>
              </td>
            </tr>
          </table> 
    </section>
</div>
```

Notice that the buttons make a call to their respective methods when you click on them.

After adding some styles and posting some messages into the database using Postman, we should get something like this.

![Result](image.png)