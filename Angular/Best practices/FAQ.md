# FAQ

## I have a class, component, or any type of data with attributes and a constructor. What should I initialize when declaring the attributes, and what should I initialize in the constructor.

The rule of thumb is that, if you are doing anything more than a simple assignement of a value to an attribute, you should do it in the constructor.

For example, let's say that you're making an Angular component that will represent a Reactive Form. You would need to initialize a _FormControl_ object for each input of the form, and a few oter simple attributes to do some other behind th scenes stuff.

It should look like this:

```ts
import { MyService } from 'src/app/Services/myService.service';
import { MyDTO } from 'src/app/Models/my.dto';

// Other imports

export class MyComponent {
  attribute1: boolean | null = null;
  attribute2: string = 'This is a string';
  attribute3: MyDTO;
  attribute4: number;

  email: FormControl;
  password: FormControl;

  loginForm: FormGroup;

  constructor(private myService: MyService, private fb: FormBuilder) {
    this.attribute3 = this.myService.getMyDTO();
    this.attribute4 = this.getNumber();

    this.email =  = new FormControl(this.attribute3.email, [
        Validators.required,
        Validators.email
    ]);

    this.password =  = new FormControl(this.attribute3.password, [
        Validators.required,
        Validators.password
    ]);

    this.loginForm = this.fb.group({
        email: this.email,
        password: this.password
    });
  }

  private getNumber() {
    return Math.random();
  }
}
```

As you can see, _attribute1_ and _attribute2_ are initialized when declared because we are only assigning a _null_ value or a string to our attributes.

Meanwhile, for _attribute3_ and _attribute4_ we need to call a method from our service or from our own component to get the value we need. We could have done this in our declarations section, but we did it in our constructor.

The same goes for our form attributes (_email, password_ and _loginForm_). We are making a call to a different constructor and other functions that require defining complex variables, such as objects. Again, we could have done this in our declarations, but we did it in our constructor because these operations are more complex than simply assigning values. 

## What are modules (custom modules) and why should I use them?
In Angular, custom modules are modules you create to organize, encapsulate, and manage specific parts of your application's functionality. They follow the same structure as Angular's core NgModule, allowing you to group related components, services, pipes, and directives together in a cohesive, reusable way.

```ts
// user.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { UserListComponent } from './user-list/user-list.component';
import { UserService } from './user.service';

@NgModule({
  declarations: [UserProfileComponent, UserListComponent],
  imports: [CommonModule],
  providers: [UserService],
  exports: [UserProfileComponent, UserListComponent] // Export if other modules need access
})
export class UserModule {}
```

Using custom modules in an Angular app helps organize code, improve maintainability, and optimize load times. Here are the main benefits:

1. **Modularity and Organization**: Custom modules let you group related components, services, pipes, and directives together, which makes your code more organized and manageable. For example, you could have a `UserModule` to handle all user-related components and services.

2. **Reusability**: When functionality is grouped in modules, you can easily import and reuse it across different parts of your app or even in other projects. This is especially useful for features like authentication, shared UI components, or custom pipes.

3. **Improved Dependency Management**: Custom modules help manage dependencies more effectively, reducing the risk of circular dependencies and making your app's structure easier to understand.

4. **Lazy Loading and Performance Optimization**: Custom modules allow you to use lazy loading, loading only the modules you need when they're required. This reduces initial load time by only loading essential code first, which is particularly beneficial in large applications.

5. **Testing and Maintainability**: Modules isolate specific parts of the application, making unit tests easier to write and maintain. Each module can be tested individually, improving the overall reliability of the app.

By creating custom modules, you make your Angular app more scalable, organized, and efficient, especially as it grows in size and complexity.
