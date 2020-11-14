---
layout: post
title:      "Redirect with React Router"
date:       2020-11-14 17:16:45 -0500
permalink:  redirect_with_react_router
---


Two ways to handle redirecting on a user event such as create, update and delete with React Router.


## Using the History API

When React Router renders a component, it passes 3 props: `history`, `match` and `location`. What we are interested in here is history.

The History library is keeping track of session history for React Router. The history prop comes with many methods and properties, including push which pushes a new entry onto the history stack. Find more on the React Router documentation [here](https://reactrouter.com/web/api/history).

Using history.push is simple:

`this.props.history.push('path')`


**Example component:**

(Github for this project is [here](https://github.com/agiletkiewicz/my-minimony-frontend))

```
import React from 'react';
import Form from 'react-bootstrap/Form';
import Button from 'react-bootstrap/Button';
import Col from 'react-bootstrap/Col';
import Row from 'react-bootstrap/Row';
import Container from 'react-bootstrap/Container';
import { connect } from 'react-redux';
import { addPost } from '../../actions/addPost';
class PostsInput extends React.Component {
constructor(props) {
        super(props);
        this.state = {
            title: "",
            description: "",
            imageUrl: "",
            url: ""
        }
    }
handleSubmit = (event) => {
        event.preventDefault();
        this.props.addPost(this.state, this.handleSuccess);
    }
handleSuccess = (postId) => {
        this.setState({
            title: "",
            description: "",
            imageUrl: "",
            url: ""
        });
        this.props.history.push(`/posts/${postId}`);
    }
handleChange = (event) => {
        this.setState({
            [event.target.name]: event.target.value
        })
    }
render() {
        return (
            <Container>
                <br />
                <Row className="justify-content-md-center">
                    <Col xs lg="5">
                        <h2>Add a new post</h2>
                        <Form onSubmit={this.handleSubmit}>
                            <Form.Label>Title</Form.Label>
                            <Form.Control 
                                type="text" 
                                name="title" 
                                onChange={this.handleChange} 
                                value={this.state.title}
                            />
                            <Form.Label>Description</Form.Label>
                            <Form.Control 
                                as="textarea" 
                                name="description" 
                                onChange={this.handleChange} 
                                value={this.state.description}
                            />
                            <Form.Label>Image URL</Form.Label>
                            <Form.Control 
                                type="text" 
                                name="imageUrl" 
                                onChange={this.handleChange} 
                                value={this.state.imageUrl}
                            />
                            <Form.Label>URL</Form.Label>
                            <Form.Control 
                                type="text" 
                                name="url" 
                                onChange={this.handleChange} 
                                value={this.state.url}
                            />
                            <br />
                            <Button 
                                variant="secondary" 
                                type="submit"
                            >
                                Submit
                            </Button>
                        </Form>
                    </Col>
                </Row>
            </Container>
        )
    }
}
const mapStateToProps = state => {
    return {
        userId: state.user.id
    }
}
export default connect(mapStateToProps, { addPost })(PostsInput);
```


In this example, I am using the Redux pattern so I pass handleSuccess to the addPost action and call it if the POST request is successful.

This is the Route:

`<Route exact path="/posts/new" render={props => (<PostsInput {...props}/>)} />`

We only have access to the history prop in components rendered by React Router. To get access to this prop in a component not rendered by React Router, we can use the withRouter higher-order component.

`import { withRouter } from 'react-router-dom'`

Add withRouter to the component and use history.push as normal.


## Using `<Redirect />` component

Using the `<Redirect />` component is arguably more in line with the principles of React.

Once you import the Redirect component from React Router, you can render the `<Redirect />` component based on changes to the local state of the component in question.

Here is an example from the same project:

```
import React from 'react';
import Button from 'react-bootstrap/Button';
import { connect } from 'react-redux';
import { deleteBoard } from '../../actions/deleteBoard';
import { Redirect } from 'react-router-dom';
class RemoveBoardButton extends React.Component {
constructor() {
        super();
        this.state = {
            redirect: false
        }
    }
handleClick = (event) => {
        event.preventDefault();
        this.props.deleteBoard(this.props.board.id, this.handleSuccess);
    }
handleSuccess = () => { 
        this.setState({ redirect: true });
    }
render() {
        if (this.state.redirect) {
            return <Redirect to='/boards' />
        }
        return (
            <Button variant="danger" onClick={this.handleClick}>Delete board</Button>
        )
    }
}
export default connect(null, { deleteBoard })(RemoveBoardButton);
```


## Conclusion
Using `history.push` requires less code and using. `<Redirect />` follows the core principle of React (state change => re-render). Which you use is ultimately up to you.
