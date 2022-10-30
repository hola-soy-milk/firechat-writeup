## Requirements
- [Firebase account](console.firebase.google.com)
- Angular CLI
- Node 16


- Open firebase
<img width="357" alt="Pasted image 20221027102042" src="https://user-images.githubusercontent.com/656318/198254257-2e7a8542-55c5-499d-8d54-3730a234a7b6.png">

<img width="357" alt="Pasted image 20221027102042" src="https://user-images.githubusercontent.com/656318/198254364-58d5d358-7416-41dc-b6a9-50c774c1b996.png">

- Google analytics not necessary
<img width="342" alt="Pasted image 20221027102247" src="https://user-images.githubusercontent.com/656318/198254405-7f79e887-acc0-499e-aea2-343c12a93ce7.png">

Under the build menu, pick Realtime Database
<img width="252" alt="Pasted image 20221027102311" src="https://user-images.githubusercontent.com/656318/198254440-8131a799-a89c-4cb0-81b5-66dae60a4f44.png">

Click on Create Database
Region close to yours
Activate test mode
<img width="796" alt="Pasted image 20221027102534" src="https://user-images.githubusercontent.com/656318/198254475-111aa5ae-8b7a-4782-9c9c-f984578ae21f.png">

Under the Rules tab, set the permissions to be always allowed (not encouraged, but okay for testing purposes!)
<img width="1231" alt="Pasted image 20221027102842" src="https://user-images.githubusercontent.com/656318/198254506-ab7ad2b0-54bc-4215-9ecc-43e717777b65.png">

Click on publish

## Angular App
- Using the angular cli, create a new app

		ng new firechat

Do not add routing.
Select CSS

Let installer finish

		cd firechat

### Install @angular/material

		ng add @angular/material

Proceed
Any option will do. I went with indigo/pink

Add typography styles
Include and enable animations

## Integrate firebase

Back in the firebase console,

<img width="457" alt="Pasted image 20221027105224" src="https://user-images.githubusercontent.com/656318/198254557-caab1997-5c19-4cab-9e6e-6fb89aef81ce.png">

Open project settings

Under General, scroll down and create a web app
<img width="1010" alt="Pasted image 20221027105309" src="https://user-images.githubusercontent.com/656318/198254590-5446bf9c-ae69-4872-943a-b95e29c5f759.png">
<img width="526" alt="Pasted image 20221027105327" src="https://user-images.githubusercontent.com/656318/198254615-401a0128-1aa6-42e7-b414-4e44e262b652.png">
<img width="777" alt="Pasted image 20221027105402" src="https://user-images.githubusercontent.com/656318/198254634-f125fc47-198d-46d5-8b2f-4f92182b4166.png">

Copy the contents of firebaseConfig:

Back in your codebase, open `src/environments/environment.ts`, and enter these into a firebase config object:

```typescript
export const environment = {
  firebase: {
      // SETTINGS FROM CONFIG
  }
  production: false
};

```

Let's install the firebase dependency:

		npm install firebase

### Setup app module

Open up `src/app/app.module.ts` and import the material tags:

```typescript
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatIconModule } from '@angular/material/icon';
import { MatCardModule } from '@angular/material/card';
import { MatDividerModule } from '@angular/material/divider';
```

Add these, as well as the form building modules to your `imports` array:

```typescript
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    FormsModule,
    ReactiveFormsModule,
    MatInputModule,
    MatIconModule,
    MatCardModule,
    MatDividerModule,
    MatFormFieldModule,
  ],

```

### Create `Chat` type

Create the file `src/chat/chat.ts`:

```typescript
export interface Chat {
  id?: string;
  username: string;
  message: string;
  timestamp: Date;
}
```

Install the uuid dependency:

		npm install uuid @types/uuid

Open up `src/app/app.component.ts`:

```typescript
import { environment } from '../environments/environment'
import { FirebaseApp, initializeApp } from 'firebase/app';
import { Database, getDatabase, ref, set, onValue  } from "firebase/database";
import { FormControl, FormGroupDirective, FormBuilder, FormGroup, NgForm, Validators } from '@angular/forms';
import { v4 as uuidv4 } from 'uuid';
import { Chat } from '../chat/chat'
```

Inside our `AppComponent` class, let's add our properties:

```typescript
  app: FirebaseApp;
  db: Database;
  form: FormGroup;
  username = 'ramonh';
  message = '';
  chats: Chat[] = [];
```

Next, let's add the constructor to our class, which sets up the realtime database and form:

```typescript
  constructor(private formBuilder: FormBuilder) {
    this.app = initializeApp(environment.firebase);
    this.db = getDatabase(this.app);
    this.form = this.formBuilder.group({
      'message' : [],
      'username' : []
    });
  }
```

Next, let's add our form submit callback to the class:

```typescript
  onChatSubmit(form: any) {
    const chat = form;
    chat.timestamp = new Date().toString();
    chat.id = uuidv4();
    set(ref(this.db, `chats/${chat.id}`), chat);
    this.form = this.formBuilder.group({
      'message' : [],
      'username' : [chat.username],
    });
  }
```

This sends off the chat object to be stored on firebase. We'll test this soon!

Finally, we'll set up our component to connect to the realtime database to load up the latest chats:

```typescript
  ngOnInit(): void {
    const chatsRef = ref(this.db, 'chats');
    onValue(chatsRef, (snapshot: any) => {
      const data = snapshot.val();
      for(let id in data) {
        if (!this.chats.map(chat => chat.id).includes(id)) {
          this.chats.push(data[id])
        }
      }
    });
  }
```

### To the view!

Open up `src/app/app.component.html`:

```html
<div id="history">
  <div *ngFor="let chat of chats" class="mb">
    <mat-card>
      <mat-card-subtitle>
        <span>{{chat.username}} (Sent: {{chat.timestamp | date:'long'}})</span>
      </mat-card-subtitle>
      <mat-card-content>
        <p>{{chat.message}}</p>
      </mat-card-content>
    </mat-card>
  </div>
</div>
<form id="message" [formGroup]="form" (ngSubmit)="onChatSubmit(form.value)">
  <mat-form-field appearance="outline" class="fw">
    <mat-label>Username</mat-label>
    <input matInput
                    formControlName="username" >
  </mat-form-field>
    <mat-form-field appearance="outline" class="fw">
    <mat-label>Enter Chat</mat-label>
      <input matInput
                      formControlName="message" >
                      <button type="submit" matSuffix mat-icon-button aria-label="Submit">
                        <mat-icon>send</mat-icon>
                      </button>
    </mat-form-field>
</form>
```

Open up `src/app/app.component.css`:

```css
#history {
  margin-left: auto;
  margin-right: auto;
  padding-left: 1em;
  padding-right: 1em;
  width: 400px;
  height:60%;
  overflow-y: scroll;
}

#message {
  margin-left: auto;
  margin-right: auto;
  width: 400px;
}

.fw {
  width: 100%
}

.mb {
  margin-bottom: 1em;
}
```

### Test the app

		ng serve --open

<img width="438" alt="image" src="https://user-images.githubusercontent.com/656318/198885253-b95169db-1c20-4585-9f27-de0c0471585f.png">


Back in the firebase console, we can see the chats get added. We can delete them here too
<img width="761" alt="Pasted image 20221027115223" src="https://user-images.githubusercontent.com/656318/198254967-efc2b14b-6d37-4ffc-949b-acf5632e75fa.png">



# Firechat

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 14.2.6.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.io/cli) page.
