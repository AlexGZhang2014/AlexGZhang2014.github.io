---
layout: post
title:      "React/Redux + Rails API: The Final Project"
date:       2018-09-16 02:00:56 +0000
permalink:  react_redux_rails_api_the_final_project
---


Wow this project took awhile to complete, but it was totally worth it. I got to use most of my existing knowledge of React and Redux that I learned over the past few weeks, and it was exciting. I knew going in that my project was going to go way beyond the minimum requirements of 2 containers and 5 stateless components, so I definitely prepared myself for the long haul. I was also a little worried about losing power (and internet) due to Hurricane Florence since I live in NC, but that didn't end up being an issue. Now, let's dive in!

The first step was setting up the Rails API. I ran the following command once I was in the file directory for my project: 
```
rails new . --api --database=postgresql -T --no-rdoc --no-ri
```
This generated a new rails app using postgres as the database (which is great if you plan to deploy your app using Heroku, which I am) and set up the API module, which I'll get to in a second.

The second step was to create the React app using ```create-react-app``` inside a ```client``` folder. Once npm finally installed everything necessary for React to work, I then set up the proxy server so that the API calls everything using the correct port.
```
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:3001",
  "dependencies": {
    ...
}
```
The important part to take note here is the proxy key that points to ```http://localhost:3001"``` - that is where the rails server lives. Setting up the proxy means that our React server and our Rails server will communicate with each other effortlessly during both development and production.

The next step was to define the ActiveModel relationships.
```
class Collection < ApplicationRecord
  has_many :items
  has_many :reviews
end

class Item < ApplicationRecord
  belongs_to :collection
end

class Review < ApplicationRecord
  belongs_to :collection
end

class Post < ApplicationRecord
  has_many :comments
end

class Comment < ApplicationRecord
  belongs_to :post
end
```

My goal was to let users create posts and then comment on them as well as create collections and fill them with items. Leaving reviews on collections is like leaving comments on posts, except that reviews also include a rating.

The next step was creating the database and seeding it. After that, I set up the routes and controller actions.
```
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :reviews
      resources :items
      resources :collections
      resources :comments
      resources :posts
    end
  end
end
```
Namespacing the routes inside /api/v1/ is considered best practices. As for controller actions, they were simple - just need to render JSON if successful.
```
module Api::V1

class CollectionsController < ApplicationController
  before_action :set_collection, only: [:show, :update, :destroy]

  # GET /collections
  def index
    @collections = Collection.order(:id)

    render json: @collections
  end

  # GET /collections/1
  def show
    render json: @collection
  end

  # POST /collections
  def create
    @collection = Collection.new(collection_params)

    if @collection.save
      render json: @collection, status: :created
    else
      render json: @collection.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /collections/1
  def update
    if @collection.update(collection_params)
      render json: @collection
    else
      render json: @collection.errors, status: :unprocessable_entity
    end
  end

  # DELETE /collections/1
  def destroy
    @collection.destroy
    if @collection.destroy
      head :no_content, status: :ok
    else
      render json: @collection.errors, status: :unprocessable_entity
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_collection
      @collection = Collection.find(params[:id])
    end

    # Only allow a trusted parameter "white collection" through.
    def collection_params
      params.require(:collection).permit(:name, :description, :owner)
    end
end

end
```
The other 4 models are set up the exact same way. Now that all controllers were set up to render JSON, navigating to ```http://localhost:3001/api/v1/posts``` returns our data in JSON format, which was helpful for determining how to render it in our React app.

Finally, we're onto creating our React components! My basic structure was basically ComponentContainer -> Components -> Component. An example is my ```PostsContainer``` renders my ```Posts``` component, which then renders each ```Post``` component. Each ```Post``` component renders its own ```CommentsContainer``` that renders ```Comments```, which finally renders ```Comment```.  The ```Collection``` line of components were set up the same way, with ```Item``` and ```Review``` components nested under each ```Collection```.

My process for each model was to first define the specific function for an action. These functions lived in a file like ```postActions.js```. These functions were responsible for fetching all the data from the Rails API. The second step was to define the case statement inside the model's reducer. All reducers came together in ```combineReducers``` so that there would only be one state in the redux store. The third step was to connect the redux store to the appropriate Container component and define its ```mapStateToProps``` and ```mapDispatchToProps``` functions. After that, I would pass down the appropriate props to each component that the container would render. Below are my finished ```Collection``` components:
```
export function fetchCollections() {
  return (dispatch) => {
    dispatch({ type: 'LOADING_COLLECTIONS' });
    return fetch('/api/v1/collections.json')
      .then(response => response.json())
      .then(collectionData => dispatch({ type: 'FETCH_COLLECTIONS', payload: collectionData }));
  }
}

export function addCollection(state) {
  return dispatch => {
    dispatch({ type: 'ADDING_COLLECTION' });

    const collectionData = {
      collection: {
        name: state.name,
        description: state.description,
        owner: state.owner
      }
    }

    fetch('/api/v1/collections', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(collectionData)
    })
      .then(response => response.json())
      .then(collectionJSON => dispatch({
        type: "ADDED_COLLECTION",
        id: collectionJSON.id,
        name: collectionJSON.name,
        description: collectionJSON.description,
        owner: collectionJSON.owner
      }));
  }
}

export function deleteCollection(id) {
  return dispatch => {
    dispatch({ type: 'DELETING_COLLECTION' });
    if (window.confirm("Are you sure?")) {
      fetch('/api/v1/collections/' + id, { method: 'DELETE' })
        .then(response => dispatch({
          type: 'DELETED_COLLECTION',
          id: id
        }))
    }
  }
}

export function updateCollection(state) {
  return dispatch => {
    dispatch({ type: 'UPDATING_COLLECTION'});

    const editedCollectionData = {
      collection: {
        id: state.id,
        name: state.name,
        description: state.description,
        owner: state.owner,
        items: state.items,
        reviews: state.reviews
      }
    }

    fetch('/api/v1/collections/' + state.id, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editedCollectionData)
    })
      .then(response => response.json())
      .then(collectionJSON => dispatch({
        type: 'UPDATED_COLLECTION',
        id: collectionJSON.id,
        name: collectionJSON.name,
        description: collectionJSON.description,
        owner: collectionJSON.owner,
        items: collectionJSON.items,
        reviews: collectionJSON.reviews
      }));
  }
}

```

```
export default function collectionsReducer(state = {
  loading: false,
  collections: [],
}, action) {
  switch (action.type) {
    case 'LOADING_COLLECTIONS':
      return { ...state, loading: true }

    case 'FETCH_COLLECTIONS':
      return { loading: false, collections: action.payload }

    case 'ADDED_COLLECTION':
      const collection = { id: action.id, name: action.name, description: action.description, owner: action.owner, items: [], reviews: [] };

      return {
        ...state,
        collections: [...state.collections, collection]
      }

    case 'DELETED_COLLECTION':
      return {
        ...state,
        collections: state.collections.filter(collection => collection.id !== action.id)
      }

    case 'UPDATED_COLLECTION':
      const editedCollection = {
        id: action.id,
        name: action.name,
        description: action.description,
        owner: action.owner,
        items: action.items,
        reviews: action.reviews
      };

      let newState = {
        ...state,
        collections: state.collections.map(collection => {
          if (collection.id !== action.id) {
            return collection;
          }
          return editedCollection;
        })
      };

      return newState;

    default:
      return state;
  }
}
```

```
import React, { Component } from 'react'
import { connect } from 'react-redux'
import Collections from '../components/collections/Collections'
import { fetchCollections, addCollection, deleteCollection, updateCollection } from '../actions/collectionActions'
import { addReview } from '../actions/reviewActions'
import AddCollectionForm from '../components/collections/AddCollectionForm'

class CollectionsContainer extends Component {
  constructor(props) {
    super(props);
    this.state = {
      editCollectionId: null
    }
  }

  toggleEditOn = id => {
    this.setState({
      editCollectionId: id
    })
  }

  toggleEditOff = () => {
    this.setState({
      editCollectionId: null
    })
  }

  componentDidMount() {
    this.props.fetchCollections()
  }

  render() {
    return (
      <div className="collections-container">
        <h1>Your Collections Feed!</h1>
        <AddCollectionForm addCollection={this.props.addCollection} />
        <Collections
          collections={this.props.collections} deleteCollection={this.props.deleteCollection}
          updateCollection={this.props.updateCollection}
          addReview={this.props.addReview}
          toggleEditOn={this.toggleEditOn}
          toggleEditOff={this.toggleEditOff}
          editCollectionId={this.state.editCollectionId}
          />
      </div>
    )
  }
}

const mapStateToProps = state => ({
  collections: state.collections.collections
})

const mapDispatchToProps = dispatch => ({
  fetchCollections: () => dispatch(fetchCollections()),
  addCollection: state => dispatch(addCollection(state)),
  deleteCollection: state => dispatch(deleteCollection(state)),
  updateCollection: state => dispatch(updateCollection(state)),
  addReview: state => dispatch(addReview(state))
})

export default connect(mapStateToProps, mapDispatchToProps)(CollectionsContainer)
```

```
import React, { Component, Fragment } from 'react'
import Collection from './Collection'
import EditCollectionForm from './EditCollectionForm'
import ItemsContainer from '../../containers/ItemsContainer'
import ReviewsContainer from '../../containers/ReviewsContainer'

class Collections extends Component {
  render() {
    return (
      <div className="collections">
        {this.props.collections.map(collection => {
          if (this.props.editCollectionId === collection.id) {
            return (
              <Fragment key={collection.id}>
                <EditCollectionForm
                  collection={collection}
                  key={collection.id}
                  updateCollection={this.props.updateCollection}
                  toggleEditOff={this.props.toggleEditOff}
                  />
                <ItemsContainer
                  collection={collection}
                  key={collection.id}
                  editCollectionId={this.props.editCollectionId}
                  />
                <ReviewsContainer
                  collection={collection}
                  key={collection.id}
                  />
              </Fragment>
            )
          } else {
            return (
              <Collection
                key={collection.id}
                collection={collection} deleteCollection={this.props.deleteCollection}
                toggleEditOn={this.props.toggleEditOn}
                addReview={this.props.addReview}
                />
            )
          }
        })}
      </div>
    )
  }
}

export default Collections
```

```
import React, { Component } from 'react'
import ItemsContainer from '../../containers/ItemsContainer'
import ReviewsContainer from '../../containers/ReviewsContainer.js'
import Moment from 'react-moment'
import 'moment-timezone'
import AddReviewForm from '../reviews/AddReviewForm'
import Button from '@material-ui/core/Button'

class Collection extends Component {
  constructor(props) {
    super(props);
    this.state = {
      addReviewStatus: false
    }
  }

  toggleAddReview = () => {
    this.setState({
      addReviewStatus: !this.state.addReviewStatus
    })
  }

  render() {
    const collection = this.props.collection

    const buttonOrForm = () => !this.state.addReviewStatus ? <Button variant="contained" color="primary" onClick={this.toggleAddReview}>Add a review</Button> : <AddReviewForm collection={collection} toggleAddReview={this.toggleAddReview} addReview={this.props.addReview}/>

    return (
      <div className="collection">
        <h2 className="collection-name">{collection.name}</h2>
        <h4>Owner: {collection.owner} (Created <Moment date={collection.created_at} fromNow />)</h4>
        <p>{collection.description}</p>
        <h6>Last updated: <Moment date={collection.updated_at} fromNow /></h6>
        <Button variant="contained" color="primary" onClick={() => this.props.toggleEditOn(collection.id)}>Edit this collection</Button>
        <Button variant="contained" color="secondary" onClick={() => this.props.deleteCollection(collection.id)}>Delete this collection</Button>
        {buttonOrForm()}
        <ItemsContainer collection={collection} />
        <ReviewsContainer collection={collection} />
      </div>
    )
  }
}

export default Collection
```

Wow that is a lot of code for just one of my models! The most important takeaways from these code snippets:
1. My containers define most of the non-standard React functions that are important for my app to work. This is because any function that a child component needs can be passed down to them thru props. In my app, the ```editCollectionId``` state is passed down via props multiple times so that it reaches the ```Item``` component! This is because my app does not allow a user to edit an item unless they click on the "Edit collection" button first. Using React's state and props makes this work surprisingly well with my buttons, components, and forms all flowing seamlessly.
2. The ```Collections``` component is stateless because all it has to do is render components - it does use props in its logic to decide whether to render an ```EditCollectionForm``` or a ```Collection``` component.
3. I used Material-UI components to give my app some style. I also did some custom CSS in ```App.css```.

Overall, this final project was definitely the toughest one to build because there were so many different "components" to make (sorry, bad pun). However, I did have a lot of fun figuring out the logic and the functions needed to connect my frontend and backend. After I pass the technical assessment, I will hopefully deploy my app to Heroku and then finally begin my job search! You can look forward to another blog post from me when week 1 of my job search arrives.
