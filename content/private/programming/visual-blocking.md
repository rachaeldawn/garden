---
title: "Code Should Read Like English"
tags:
  - programming
  - clean-code
---


Compare the two code snippets ...


```typescript
async function register(email: string, password: string): Promise<void> {
  // verify email
  if (/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/.test(email) !== true) {
    alert('invalid email provided!')
    return
  }

  // password length check
  if (password.length < 8) {
    alert('password must be at least 8 characters');
    return
  }

  // password is "secure"
  if (
    /\w/.test(password) !== true
    && /[A-Z]/.test(password) !== true
    && /[0-9]/.test(password) !== true
    || /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password) !== true
  ) {
    alert('Password must have at least 1 upper case, 1 lower case, 1 number, and 1 special character');
    return
  }

  // user isn't already logged in
  if (localStorage.get('app:login-token')) {
    // open the modal saying they're already logged in
    alert('You are already logged in!');
    return
  }

  const response = await http.post('https://my-api.dev.example.com/register', { body: { email, password } });
  
  if (response.body.code === 409) {
    alert('Email already in use!')
  } else if (response.body.code === 400) {
    alert('Uh oh! You appear to have not provided a valid email. Did you mean to log in?');
  } else if (response.body.code === 200) {
    alert('Registration success! Please check your email');
  } else {
    console.log('Unhandled code received', response.body.code);
  }

  // assume page is https://my-site.com/register
  window.location = window.location.search = "success=true";
}

```

```typescript
const loginTokenKey = 'app:login-token';
async function register(email: string, password: string): Promise<void> {
  const validationErrors = validateRegistration(email, password);
  if (validationErrors.length > 0) {
    handleRegistrationErrors(validationErrors);
    return;
  }

  const response = await http.post('https://my-api.dev.example.com/register', { body: { email, password } });
  
  if (response.body.code === 409) {
    alert('Email already in use!')
  } else if (response.body.code === 400) {
    alert('Uh oh! You appear to have not provided a valid email. Did you mean to log in?');
  } else if (response.body.code === 200) {
    alert('Registration success! Please check your email');
  } else {
    console.log('Unhandled code received', response.body.code);
  }

  // assume page is https://my-site.com/register
  window.location = window.location.search = "success=true";
}

function handleRegistrationErrors(errors: string[]): void {
  const elements: HTMLElement[] = errors.map(msg => document.createElement('p'));

  for (let i = 0; i < errors.length; ++i) {
    const el = elements[i];
    const err = errors[i];

    el.classList.add('error-message');
    el.textContent = err;
  }

  // assume it exists
  const container = document.getElementById('error-container');
  container.textContent = ''; // clear other errors
  container.append(...elements);
}

function validateRegistration(email: string, password: string): string[] {
  return [ ...validateEmail(email), ...validatePassword(password) ];
}

/**
 * @return {string[]} An array of errors to show to the user
 */
function validateEmail(email: string): string[] {
  if (/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/.test(email)) {
    return [];
  }


  return ['Invalid email'];
}

function validatePassword(password: string): string[] {
  const tests = {
    hasOneLowercase: {
      check:    /[a-z]/,
      message: 'Needs to have at least one lowercase character',
    },
    hasOneUppercase: {
      check:    /[A-Z]/,
      message: 'Needs to have at least one uppercase character',
    },
    minimumEightCharacters: {
      check:    /.{8,}/,
      message: 'Needs to be at least 8 characters long'
    },
    oneSpecialCharacter: {
      check:    /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/,
      message: 'Needs to have at least one special character',
    }
  };

  const errors: string[] = [];

  for (const [ name, { check, message } ] of Object.values(tests)) {
    if (check.test(password)) {
      continue;
    }

    console.warn(`${name} failed validation`);
    errors.push(message);
  }

  return errors;
}

function isLoggedIn(): boolean {
  const token = getLoginToken();
  return token?.length > 0;
}

function getLoginToken(): string | null {
  return localStorage.get(loginTokenKey);
}
```