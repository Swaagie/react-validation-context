# react-validation-context

Components for providing validation via React context.

# Install

This package is available on `npm`. Install it using:

```sh
npm install react-validation-context --save
```


# Usage

This library revolves around the idea of "validity". A component can have one of the following validities:

- `@typedef {(null|Boolean)} Validity`
  - `null` - Validation is disabled.
  - `true` - Validation passed.
  - `false` - Validation failed.

It is useful to know when a component's validity changes. As such, this library attempts to provide a uniform API for validation
change handlers. In general, a validity change handler has the following API:

- `{Function} onValidChange` - Validity change handler.
  - `@param {String} name` - The unique identifier for the component whose validity changed.
  - `@param {Validity} isValid` - The current validity of the component.
  - `@param {Validity} wasValid` - The previous validity of the component.

## `Validates`

The `Validates` component is used to wrap a component that can be validated, providing the logic for validation change handlers.

### Props

- `{String} name` - A unique identifier for the component.
- `{Validity} validates` - The component's validity.
- `{Function} onValidChange` - Validity change handler.
- `{ReactElement} children` - Children. The component only accepts a single child, and will simply render as that child.

### Context

If `onValidChange` is present in `Validate`'s context, it will call that context handler appropriately.

### Example usage

```jsx
import React from 'react';
import { Validates } from 'react-validation-context';

export default class RequiredInput extends React.Component {
  constructor (props) {
    super(...arguments);

    // Set up the initial state based on whether the initial value validates
    const { value, defaultValue } = props;
    this.state = { validates: this.validate(value || defaultValue) };
  }

  validate (val) {
    // Check that the value exists and has non-whitespace content
    return val && val.trim().length > 0;
  }

  render () {
    const { onChange: origOnChange, onValidChange, name, children, ...rest } = this.props;
    const { validates } = this.state;

    // Wrap the onChange handler to update `this.state.validates`
    const onChange = (e) => {
      if (origOnChange) {
        origOnChange(e);
      }

      this.setState({ validates: this.validate(e.target.value) });
    };

    // Render `input` and validation context-aware `Validates`
    return <Validates validates={validates} onValidChange={onValidChange} name={name}>
      <label>
        <input type="text" onChange={onChange} name={name} {...rest} />
        {children}
        {validates ? null : 'This input is required'}
        </label>
    </Validates>;
  }
}

RequiredInput.propTypes = {
  name: React.PropTypes.string.isRequired, // Input identifier name
  value: React.PropTypes.string, // Input value
  defaultValue: React.PropTypes.string, // Default input value
  onChange: React.PropTypes.func, // onChange handler for input
  onValidChange: React.PropTypes.func, // validity change handler
  children: React.PropTypes.node // React children
};

RequiredInput.defaultProps = {
  validate: () => null // By default, validation is disabled, so return `null`
};
```


## `Validate`

The `Validate` component is used to wrap a component which has descendants that may be validated, and provides an interface for
validating all of those descendants. It extends `Validates` to provide the same interface for listening for validation changes on
the component itself.

**NOTE**: This component is able to keep track of all conforming descendant components (not just direct children) via the [React
`context` api][react-docs-context].

### Props

- `{Function} validate` - Validation function for descendants.
  - `@param {Object} valids` - The keys are the descendant `name`s, and the values are their `{Validity}`s.
  - `@returns {Validity}` - The validity of this component.

This component also inherits all the props of `Validate`.

### Context

If `onValidChange` is present in `Validate`'s context, it will call that context handler appropriately.

### Example usage

```jsx
import React from 'react';
import { Validate } from 'react-validation-context';

export default class Form extends React.Component {
  constructor () {
    super(...arguments);

    // The form is initially valid
    this.state = { validates: true };
  }

  render () {
    const { onValidChange: origOnValidChange, onSubmit: origOnSubmit, children, name, className, ...rest } = this.props;
    const { validates } = this.state;

    // The form is invalid if there are any invalid items in its validation context
    const validate = valids => Object.keys(valids).every(k => valids[k] !== false);

    // Wrap the onValidChange handler to set this.state.validates
    const onValidChange = (name, isValid, wasValid) => {
      if (origOnValidChange) {
        origOnValidChange(name, isValid, wasValid);
      }

      this.setState({ validates: isValid });
    }

    // Wrap the onSubmit handler to prevent submission if the form is invalid
    const onSubmit = e => {
      if (origOnSubmit) {
        origOnSubmit(e);
      }

      if (!this.state.validates) {
        e.preventDefault();
      }
    }

    // Render `form` and create validation context `Validate` (which is also validation context-aware)
    return <Validate validate={validate} onValidChange={onValidChange} name={name}>
      <form onSubmit={onSubmit} name={name} {...rest}>
        {children}
      </form>
    </Validate>;
  }
}

Form.propTypes = {
  name: React.PropTypes.string.isRequired, // Form identifier name
  onSubmit: React.PropTypes.func, // onSubmit handler for form
  onValidChange: React.PropTypes.func, // validity change handler
  children: React.PropTypes.node // React children
};
```


# Demo

For a more in-depth demonstration, see the example project under `demo/`.


# Tests

Run the [`mocha`][mocha] unit tests via:

```sh
npm test
```

Coverage reports are generated using [`istanbul`][istanbul] for [Cobertura][cobertura].


[react-docs-context]: https://facebook.github.io/react/docs/context.html (React context API docs)
[mocha]: http://mochajs.org/ (mocha)
[istanbul]: https://www.npmjs.com/package/istanbul (istanbul)
[cobertura]: https://cobertura.github.io/cobertura/ (Cobertura)

