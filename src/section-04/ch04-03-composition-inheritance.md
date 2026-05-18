# Composition Over Inheritance

Before React, I spent a lot of time with object-oriented languages where inheritance was the go-to solution. Need a special button? Extend Button. Need an admin user? Extend User. It felt natural.

Then React came along and said: prefer composition over inheritance. At first, I did not get it. But after building a few projects, it clicked.

## The Problem with Inheritance

Inheritance creates tight coupling between parent and child classes. When you extend a component, you inherit everything, even stuff you do not want.

```jsx
// Inheritance approach: fragile and rigid
class Button {
  render() {
    return <button>{this.props.label}</button>;
  }
}

class IconButton extends Button {
  render() {
    return (
      <button>
        <Icon name={this.props.icon} />
        {this.props.label}
      </button>
    );
  }
}
```

What if you want a button with just an icon and no label? What if you want the icon after the text? You end up overriding more and more, and the hierarchy gets messy.

## Composition: Has-A vs Is-A

Inheritance is "is-a." A Dog is an Animal. Composition is "has-a." A Car has an Engine.

React favors "has-a." Components contain other components rather than extending them.

```jsx
// Composition approach: flexible and reusable
function Button({ children, ...props }) {
  return <button {...props}>{children}</button>;
}

// Use it however you want
<Button>Click me</Button>
<Button><Icon name="save" /> Save</Button>
<Button><Icon name="delete" /> Delete</Button>
```

The Button does not need to know what is inside it. It just wraps content in a button element. That is composition.

## The React Philosophy

The React docs explicitly say that composition solves the problems people try to solve with inheritance. Here are the common scenarios:

**Specialization:** Instead of extending a component, pass different props.

```jsx
function Dialog({ title, message, onConfirm }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <p>{message}</p>
      <button onClick={onConfirm}>OK</button>
    </div>
  );
}

function ConfirmDialog(props) {
  return <Dialog {...props} title="Are you sure?" />;
}

function ErrorDialog(props) {
  return <Dialog {...props} title="Something went wrong" />;
}
```

**Containment:** Instead of knowing what children to render, accept them.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

// Any content fits inside
<Card>
  <h3>User Profile</h3>
  <p>Name: Alex</p>
</Card>
```

## When I Finally Understood

The moment it clicked for me was when I tried to build a modal system with inheritance and got stuck in a nightmare of overrides. I rewrote it with composition in half the code and it worked beautifully. Composition gives you flexibility without the fragility.
