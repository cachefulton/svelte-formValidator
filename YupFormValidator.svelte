<script lang="ts">
  // this file depends on data-form-validator attribute on form elements that correspond to yup object keys
  // formData keys should also correspond to yup object keys with exactness


  import { onMount, type Snippet } from 'svelte';
  import { ObjectSchema, ValidationError } from 'yup'

  interface FormValidatorOptions {
    // Yup schema object
    yupSchema: ObjectSchema<any> 

    // Form data as state bound to corresponding form elements
    formData: Record<string, any> 

    // Submission handler function without form validation
    onsubmit: (formData: Record<string, any>) => Promise<any> | void 

    // this is passed automatically from parent if parent's form is wrapped in this file's import element
    children: Snippet 

    // default: 'error-message', pass your own class name for error messages that will display under each invalid form element
    errorMessageClass?: string 

    // default: 'invalid', pass your own class name for invalid form elements to add things like a red border
    invalidFormElementClass?: string 
  }

  let { 
    yupSchema, 
    formData, 
    onsubmit, 
    children, 
    errorMessageClass='error-message', 
    invalidFormElementClass='invalid' 
  }: 
  FormValidatorOptions = $props()

  let errorMessages: Record<string, string> = $state({})

  onMount(() => {
    //add submit event listener
    let formElement = document.querySelector('form')
    if (!formElement) {
      throw new Error('No form element found')
    }
    formElement.addEventListener('submit', handleSubmit)

    //add blur event listeners
    const validationElements = formElement.querySelectorAll('[data-form-validator]')
    if (!validationElements.length) {
      throw new Error('No form elements found with data-form-validator attribute')
    }
    else if (validationElements.length !== Object.keys(yupSchema.describe().fields).length) {
      throw new Error(
        'Number of form elements with data-form-validator attribute does not match number of yup object keys'
      )
    }
    validationElements.forEach((el: Element) => {
      let element = el as HTMLElement
      element.addEventListener('blur', handleBlur)
      const newSibling = document.createElement('span')
      newSibling.className = errorMessageClass
      // add an effect to add classes and set the right error message text to newSibling
      // it doesn't work with $derived() from what I can tell
      $effect(() => {
        let formValidator = element.dataset.formValidator!
        newSibling.textContent = errorMessages[formValidator]
        // check if error messages exist
        if (!errorMessages[formValidator] || errorMessages[formValidator] === '') {
          element.classList.remove(invalidFormElementClass)
        }
        else {
          element.classList.add(invalidFormElementClass)
        }
      })
      element.after(newSibling)
    })
  })

  async function handleBlur(event: FocusEvent) {
    let target = event.target as HTMLElement
    // I am confident target.dataset.formValidator exists because of how I added handleBlur event listener
    let fieldName = target.dataset.formValidator!
    try {
      await yupSchema.validateAt(fieldName, { [fieldName]: formData[fieldName] })
      errorMessages[fieldName] = ''
    } catch (error) {
      let err = error as ValidationError
      errorMessages[fieldName] = err.message
    }
  }

  async function handleSubmit(event: SubmitEvent) {
    event.preventDefault()
    try {
      await yupSchema.validate(formData, {abortEarly: false})
      await onsubmit(formData)
    } catch (error) {
      if (error instanceof ValidationError) {
        errorMessages = error.inner.reduce((acc, err) => {
          return { ...acc, [err.path as string]: err.message }
        }, {});
      }
      else {
        console.error(error)
      }
    }
  }
</script>

{@render children()}