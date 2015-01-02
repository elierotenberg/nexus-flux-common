Nexus Flux
==========

Abstract Nexus Flux Diagram
```
    +-> Action.dispatch ---+--> Client.Events ---+--> Action.onDispatch -+
    |                                                                    |
Component logic         Adapter               Adapter               Global logic
    |                                                                    |
    +-- Store.onUpdate  <--+--- Server.Events <--+--- Store.update ------+
```


Local Nexus Flux Diagram
```
Component #1 <---------+
                       |
Component #2 <---------+---> Global logic
                       |
Component #3 <---------+

```

Over the wire Nexus Flux Diagram
```
Component #A1 <---+
                  |
Component #A2 <---+-- Client #A Adapter -+
                  |                      |
Component #A3 <---+                      |
                                         +-> Global logic
Component #B1 <---+                      |
                  |                      |
Component #B2 <---+-- Client #B Adapter -+
                  |
Component #B3 <---+
```

#### Client usage example (in a React view)

```js
{
  getInitialState() {
    this.lifespan = new Promise((resolve) => this._lifespan = resolve);
    return {
      todoList: this.props.flux.Store('/todo-list').value,
    };
  }

  componentWillMount() {
    const flux = this.props.flux.within(this.lifespan);
    flux.Store('/todo-list')
    .onChange((todoList) => this.setState({ todoList }))
    .onDelete(() => this.setState({ todoList: undefined }));
    this.removeItem = flux.Action('/remove-item').dispatch;
  }

  componentWillUnmount() {
    this._lifespan();
  }

  render() {
    return this.state.todoList ? todoList.map((item, name) =>
      <div onClick={() => this.removeItem({ name })}>
        {item.get('description')} (Click to remove)
      </div>
    ) : null;
  }
}
```

#### Server usage example

```js
const todoList = server.Store('/todo-list');
const removeItem = server.Action('/remove-item');

todoList
.set('first', { description: 'My first item' })
.set('second', { description: 'My second item' })
.set('third', { description: 'My third item' })
.commit();

removeItem.onDispatch((clientID, { name }) => {
  todoList.delete(name).commit();
});
```

#### Implementation example: local service

```js
const data = {};

// provide trivial fetch/publish implementations
function fetch(path) {
  return Promise.resolve(data[path].head);
}

function publish(path, value) {
  data[path] = value;
}

// instanciate a server, a client, and a link
const server = new Server(publish);
const client = new Client(fetch);
const link = server.createLink();

// pipe both ends
client.pipe(link);
link.pipe(client);

React.render(<MyComponent flux={client} />, document.getElementById('app-root'));
```

