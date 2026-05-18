# Compound Components

Compound components are my favorite React pattern. They let you build components that work together like a team, sharing state implicitly while giving the user full control over rendering.

## The Problem

Imagine building a Tabs component. You could do it with a single prop-heavy component:

```jsx
<Tabs
  items={[
    { label: 'Tab 1', content: <p>Content 1</p> },
    { label: 'Tab 2', content: <p>Content 2</p> },
  ]}
/>
```

This works, but it is rigid. What if you want custom tab styling? What if some tabs have icons? You end up adding more and more props.

## The Compound Solution

With compound components, each piece is its own component, but they share state behind the scenes:

```jsx
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);

  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="flex border-b">{children}</div>;
}

function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  const isActive = activeIndex === index;

  return (
    <button
      className={`px-4 py-2 ${isActive ? 'border-b-2 border-blue-500' : ''}`}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

function TabPanel({ index, children }) {
  const { activeIndex } = useContext(TabsContext);
  return activeIndex === index ? <div className="p-4">{children}</div> : null;
}
```

Now the usage is clean and flexible:

```jsx
<Tabs defaultIndex={0}>
  <TabList>
    <Tab index={0}>Profile</Tab>
    <Tab index={1}>Settings</Tab>
    <Tab index={2}>Notifications</Tab>
  </TabList>
  <TabPanel index={0}><ProfileForm /></TabPanel>
  <TabPanel index={1}><SettingsForm /></TabPanel>
  <TabPanel index={2}><NotificationPrefs /></TabPanel>
</Tabs>
```

## Accordion Example

The same pattern works for Accordions:

```jsx
function Accordion({ children }) {
  const [openIndex, setOpenIndex] = useState(null);
  return (
    <AccordionContext.Provider value={{ openIndex, setOpenIndex }}>
      <div className="divide-y">{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ index, title, children }) {
  const { openIndex, setOpenIndex } = useContext(AccordionContext);
  const isOpen = openIndex === index;

  return (
    <div>
      <button onClick={() => setOpenIndex(isOpen ? null : index)}>
        {title} {isOpen ? '-' : '+'}
      </button>
      {isOpen && <div className="p-4">{children}</div>}
    </div>
  );
}

// Usage
<Accordion>
  <AccordionItem index={0} title="What is React?">
    <p>A JavaScript library for building user interfaces.</p>
  </AccordionItem>
  <AccordionItem index={1} title="What are hooks?">
    <p>Functions that let you use state in functional components.</p>
  </AccordionItem>
</Accordion>
```

## Why Compound Components Rock

- **Flexible API:** Users compose components however they want
- **Shared state:** Components communicate through context without prop drilling
- **Readable JSX:** The markup describes the structure clearly
- **Extensible:** Easy to add new pieces without changing existing ones

This pattern is used in real libraries like Radix UI and Headless UI. Once you understand it, you will see it everywhere.
