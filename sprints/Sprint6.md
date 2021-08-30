## Sprint 6: Editing and Updating Todos

### Implementing Edit

To update a `todo` in our database we will need to initiate a pre-filled form that contains that specific data.  Once the user has updated the text in the form, they can click a button to initiate a save. The actual save button will trigger the database call to update.  

In `containers/TodosContainer.js`:

```js
  state = {
    todos: [],
  };

  /* createTodo = ...  */

  /* deleteTodo = ...  */

  updateTodo = todo => {
    const isUpdatedTodo = t => {
        return t._id === todo._id;
    };

    TodoModel.update(todo)
        .then((res) => {
          let todos = this.state.todos;
          todos.find(isUpdatedTodo).body = todo.body;
          this.setState({ todos: todos });
        });
  };

  render() {
    return (
      <div className="todosComponent">
        <CreateTodoForm
          createTodo={ this.createTodo }
        />
        <Todos
          todos={ this.state.todos }
          updateTodo={ this.updateTodo } 
          deleteTodo={ this.deleteTodo }
          />
      </div>
    );
  };
```

In the `components/Todos.js`, add `updateTodo` to `<Todo>` props:

```js
//....
 let todos = props.todos.map((todo) => {
      return (
        <Todo
          key={todo._id}
          todo={todo}
          deleteTodo={props.deleteTodo}
          updateTodo={props.updateTodo} 
          />
      );
    });
//...
```

<!-- Todo changes -->
In `components/Todo.js` We need to add some state and add the method  `toggleBodyForm`:

```js
  state = {
      formStyle: {
        display: 'none',
      },
  };

  toggleBodyForm = () => {
    this.state.formStyle.display === 'block'
    ? this.setState({ formStyle: {display: 'none'} })
    : this.setState({ formStyle: {display:'block'} });
  };
```

This will hide the `todo` body and reveal the `todoForm` components.

Lets update our `Todo` render to have the `TodoForm` included. We'll also add an Edit link. When the user clicks on the edit link, the form will appear prepopulated with the text of the todo for easy altering. Neat!

```js
  render() {
    return (
      <li data-todos-index={this.props.todo._id}>
        <div>
          <span className="todo-item">
            {this.props.todo.body}</span>
          <span
            className='edit' 
            onClick={this.toggleBodyForm}>
            Edit
          </span>
          <span
            className='remove' 
            onClick={this.deleteClickedTodo}>
            Remove
          </span>
        </div>
        <TodoForm 
          todo={this.props.todo}
          style={this.state.formStyle}
          autoFocus={true}
          buttonName="Update Todo!"
          updateTodo={this.props.updateTodo}
          toggleBodyForm={this.toggleBodyForm} />
      </li> 
    );
  };
```

You will then have to both write the `TodoForm` component and then import it into `components/Todo.js`:

```js
//TodoForm.jsx
import { Component } from 'react';

class TodoForm extends Component {
  state = {
    todo: '',
  };

  onChange = (event) => {
    this.setState({
      todo: event.target.value,
    });
  };

  onSubmit = (event) => {
    event.preventDefault();
    const todo = this.props.todo;
    todo.body = this.state.todo;
    this.props.updateTodo(todo);
    this.setState({ todo: '' });
    this.props.toggleBodyForm();
  };

  render() {
    return (
      <div style={this.props.style} className='todoForm'>
        <form onSubmit={ this.onSubmit }>
          <input
            autoFocus={this.props.autoFocus}
            onChange={ this.onChange }
            placeholder='Write a todo here ...'
            type='text'
            value={this.state.todo} />
          <button type='submit'>Save</button>
        </form>
      </div>
    );
  };
};

export default TodoForm;

```

```js
//Todo.jsx
import { Component } from 'react';
import TodoForm from './TodoForm';

//...
```

In `models/Todo.js` add our method:

```js
  static update = (todo) => {
    let request = axios.put(`${endPoint}/${todo._id}`, todo);
    return request;
  };
```

Think back to what we did for the other CRUD actions--we define some axios behavior in `/models/Todo.js`. Then we define a method in `TodosContainer` that will handle update behavior.

Then we make our way down from `TodosContainer` to `Todos` to `Todo`, with `state` trickling down as `props`.

## Conclusion

We've learned how to do full CRUD for a basic todo app here. We've seen in particular how props can be trickled down through parent and child components to make a very modular app. We've also been introduced to the magic of axios for network calls from our frontend.
