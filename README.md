# svelte-formValidator
I was tired of adding error messages and validation to my forms. So I made something really simple.
This adds error messages, submit handers, and focus events


### Instructions
This depends on yup objects to validate a form.
This depends on data-form-validator attributes on form elements that correspond to yup object keys.
There should be some form data that also corresponds to yup object keys with exactness.
Pass these things as props to YupFormValidator import element
```js
  // Yup schema object
  yupSchema: ObjectSchema<any> 

  // Form data as state bound to corresponding form elements
  formData: Record<string, any> 

  // Custom submission function without form validation
  onsubmit: (formData: Record<string, any>) => Promise<any> | void 

  // this is passed automatically from parent if parent's form is wrapped in this file's import element
  children: Snippet 

  // default: 'error-message', pass your own class name for error messages that will display under each invalid form element
  errorMessageClass?: string 

  // default: 'invalid', pass your own class name for invalid form elements to add things like a red border
  invalidFormElementClass?: string 
```

#### Example

```svelte
<script lang="ts">
  import YupFormValidator from './YupFormValidator.svelte'
  import * as yup from 'yup'


  let schema = yup.object().shape({
    firstName: yup.string().required('Please enter your first name'),
    lastName: yup.string().required('Please enter your last name'),
    email: yup.string().email('Please enter a valid email').optional(),
    phone: yup.string().test(
      'is-valid-phone',
      'Please enter a 10 digit phone number',
      value => !value || /^\d{10}$/.test(value)
    )
  })
  
  let formData: Record<string, string> = $state({
    firstName: '',
    lastName: '',
    email: '',
    phone: ''
  })

 function onsubmit() {
    //just clears form data
    for (let key in formData) {
      formData[key] = ''
    }
  }
</script>

{#snippet textInput(label: string, validation: string, extraProps: object|null = null)}
  <label>
    {label}
    <input {...extraProps} type="text" data-form-validator="{validation}" bind:value={formData[validation]}/>
  </label>
{/snippet}

<YupFormValidator yupSchema={schema} formData={formData} onsubmit={onsubmit}>
  <form>
    <h1>Find</h1>
    {@render textInput('First Name', 'firstName')}
    {@render textInput('Last Name', 'lastName')}
    {@render textInput('Email', 'email', )}
    {@render textInput('Phone', 'phone', {maxlength:"10"})}
    <button type="submit">Submit</button>
  </form>
</YupFormValidator>

<style>
  .invalid {
    border-color: red;
    outline: none;
  }
  
  .error-message {
    color: red;
    font-size: 0.85rem;
    margin-top: 0.25rem;
  }
</style>
```


