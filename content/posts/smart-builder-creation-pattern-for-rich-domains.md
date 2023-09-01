---
title: "Smart Builder Creation Pattern for Rich Domains"
date: 2023-09-01T22:02:42+01:00
---

# Smart Builders
### A creation pattern for rich domains
When working in a domain-driven system, your classes (or types, more generally) become a key piece of self-documenting behaviour. Not only do rich domain objects tell you what they are, but hopefully something about why they are there at all. When the domain you’re working in has a strong need to validate these objects it might be desirable to do so as they are created, keeping significance of that idea high. However, the classic tools available in an OO language don’t always offer scalable and maintainable solutions.

## When Constructors Go Bad
The constructor is probably the first tool we would reach for here, however long argument lists and only having exceptions for flow control make them unwieldy when big enough. Cohesion is naturally very high, but this isn’t going to outweigh the maintenance cost as domain objects grow large or validation rules become complex. A naive but believable example might look something like this:

``` java
public class ContactDetails {

    private final String email;
    private final String telephoneNumber;

    public ContactDetails(String email, String telephoneNumber) {
        if (!isValidEmail(email))
            throw new ValidationException("Email is not valid!");

        if (!isValidTelephone(telephoneNumber))
            // Not checked if Email is invalid
            throw new ValidationException("Telephone number is not valid!");
        
        this.email = email;
        this.telephoneNumber = telephoneNumber;
    }
    ...
}
```

And when we come to use it, we might resort to this:

``` java
public class ContactDetailsService {

    private ContactDetailsRepository contactDetailsRepository;

    public String saveContactDetails(String email, String telephoneNumber) {

        try {
            ContactDetails contactDetails = new ContactDetails(email, telephoneNumber);
            contactDetailsRepository.save(contactDetails);
            return "Contact Details saved";
        } catch (ValidationException ve) {
            return ve.getMessage();
        }
    }
}
```

At most, only one validation rule is caught and reported! Using the constructor here is terse but the try-catch can get messy if we want to do further processing here. We would have to get creative if we wanted more detailed validation error reporting and we would lose that benefit of brevity. Overusing such low-level structures here can muddle the message when handling the higher-level ideas we’re working with here.

## Improving on Exceptions
Since failure is an expected outcome some of the time, this is a good candidate for replacing the throwing of an exception with the Notification Pattern. As it is this still isn’t quite what we are after, so we will need to adjust it slightly. The idea of accumulating all of the validation errors in a meaningful object is the key aspect.

The example use of [notification](https://martinfowler.com/eaaDev/Notification.html) pattern as described by Martin in that article depends on validation happening after the object is created so it is implemented in a public method. Since we want to catch these errors before they have any chance of representing invalid state, we have to preempt instantiation in some way.

In the meantime, we can “throw” the structured Notification object so as not to break the existing contract with the constructor.

``` java
public ContactDetails(String email, String telephoneNumber) {
    Notification notification = new Notification();
    if (!isValidEmail(email))
        notification.addError("Email is not valid!");

    if (!isValidTelephone(telephoneNumber))
        notification.addError("Telephone number is not valid!");

    if (notification.hasErrors())
        throw new ValidationException(notification);

    this.email = email;
    this.telephoneNumber = telephoneNumber;
}
```

## Applying a Creation Pattern
The next step to eliminating the need for exceptions would be to move from the constructor to a creation pattern, such as a Factory function or a Builder. Using a Factory function will be sufficient in many cases, but since this is intended for use in potentially complex or detailed domains I will skip straight to implementing a Builder.

``` java
    public static class CDBuilder {

        private String email;
        private String telephoneNumber;

        public CDBuilder withEmail(String email) { ... }

        public CDBuilder withTelephoneNumber(String telephoneNumber) { ... }

        public ContactDetails build() {
            return new ContactDetails(email, telephoneNumber);
        }

        public Notification validate() {
            Notification notification = new Notification();
            if (!isValidEmail(email))
                notification.addError("Email is not valid!");

            if (!isValidTelephone(telephoneNumber))
                notification.addError("Telephone number is not valid!");

            return notification;
        }
    }
```

As you can see, this is a conventional Builder pattern implementation but with an added validate() method. This is where we implement the validation logic, returning a Notification of any errors found in the process. The usual encapsulation tricks apply here meaning we can hide away the (now unsafe) constructor from the outside world. With this done we can check and act on any errors before we call build() and use the final object.

``` java
    public void saveContactDetails(String email, String telephoneNumber) {

        ContactDetailsBuilder builder = ContactDetails.builder()
            .withEmail(email)
            .withTelephoneNumber(telephoneNumber);

        Notification validationResult = builder.validate();

        if (validationResult.hasErrors())
            notifyError(validationResult); // A fictitious error-handling path
        else
            contactDetailsRepository.save(builder.build());
    }
```

The build method is not protected from misuse, however. Any unknowing developer might try to build the object without validating first which would result in invalid state again. This is tricky to do at compile time, but it does now truly represent unexpected runtime behaviour and can be addressed by throwing an exception if someone attempts to build() when there are errors. This should therefore be detectable in tests.

``` java
    public ContactDetails build() {
        Notification validation = this.validate();
        if (validation.hasErrors()) throw new ValidationException(validation);

        return new ContactDetails(email, telephoneNumber);
    }
```

## Making things Explicit
The pattern described here works well, but critics might point out that the meaning of the Builder has changed in this context. This may not actually pose a problem in a real project with all the right communication mechanisms in place, but it does somewhat go against the original intent of the pattern to be a self-describing creation pattern. This can be resolved, if need be, by putting a name to this intermediate state between validation and instantiation. In some sense we are “proposing” a new object, and checking if it is allowed to exist before trying to use it. Nothing can be done with this “proposed” object except validating and accepting (or rejecting) it.

``` java
// A generic class for our intermediary state
public class Proposed<T> {

    private final Notification validationResult;
    private final T value;

    public Proposed(Notification validationResult, T value) {
        this.validationResult = validationResult;
        this.value = value;
    }

    public T approved() throws ValidationException {
        if (this.hasErrors())
            throw new ValidationException(this.validationResult);
        else
            return value;
    }

    public boolean hasErrors() {
        return this.validationResult.hasErrors();
    }

    public Notification errors() {
        return validationResult;
    }
}

…
    public static class ContactDetailsProposer {
      ...
        public Proposed<ContactDetails> propose() {
            return new Proposed<ContactDetails>(
                this.validate(),
                new ContactDetails(this.email, this.telephoneNumber)
            );
        }
    }
```

Now using it looks something like this.

``` java
public class ContactDetailsService {
  ...
    public void saveContactDetails(String email, String telephoneNumber) {
        Proposed<ContactDetails> proposed = ContactDetails.propose()
            .withEmail(email)
            .withTelephoneNumber(telephoneNumber)
            .propose();

        if (proposed.hasErrors())
            notifyError(proposed.errors());
        else
            contactDetailsRepository.save(proposed.approved());
    }
}
```

As you can see this adds a not-insignificant amount of extra code, like many complex OO patterns, so this should be used with full consideration of the downsides. Make sure this is a meaningful concept to add to your system, not just in terms of code but in terms of the business rules and use cases. When that is the case though, this pattern does a good job of protecting from bad state whilst explaining itself to the reader.


