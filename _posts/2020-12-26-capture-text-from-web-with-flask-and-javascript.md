---
title: Capture text from web with flask and JavaScript
slug: capture-text-from-web-with-flask-and-javascript
date_published: 2020-12-26T12:03:02.000Z
date_updated: 2020-12-26T12:03:02.000Z
tags: python, flask, browser, extension, sqlalchemy, sideproject, ideas
---

We read a lot of stuff from web and sometimes would like to make a note of some of it so that we can refer to it later. There are a few products which charge few $/month and provide this service but we can easily write a browser extension to capture the text on a web page and save that to a database. Run this service locally if you need this on a single machine, on a raspberry pi in your home network or use a Amazon Lightsail. whatever suits you!!!

Here, I'm going to write a *basic browser extension and a web service* so that we can save the notes.

### Browser extension

This is the first component which captures the highlights on a web page. Since browser doesn't exposes any event which *directly *gives you information about highlighted text , we are going to use *mouseup *event and *getSelection* window function for that.

Once you highlight a block of text (and release mouse's left button), check if there is any highlighted text available. If yes, insert a button element in the page and attach a handler to its click event. That's all this extension does.

    function getSelected() {
        if (window.getSelection) {
            return window.getSelection();
        }
        else if (document.getSelection) {
            return document.getSelection();
        }
        else {
            var selection = document.selection && document.selection.createRange();
            if (selection.text) {
                return selection.text;
            }
            return false;
        }
    }

capture the selected text from a page
    btnSave = $('<button>')
                .attr({
                    type : 'button',
                    id : 'btnsave'
                })
                .html('+')
                .css({
                    'color' : 'red'
                }).hide();
                $(document.body).append(btnSave);

inject a button to html body
    $('#btnsave').click(function  save(event) {
                    event.preventDefault();
                    var txt = $.trim(getSelected());
                    // make an api call to save selected text
        		//...
                    document.getElementById("btnsave").style.display="none";
                });

handler for button click
The code for this extension is available at [here](https://github.com/kapilgarg/web-note/tree/master/extension/src). You should be able to install the extension locally.

### Web API

Browser extension calls this Web service to read or write . I'm using flask to write a web api which takes a block of text and saves it in database. The database I'm using here is Sqlite and SQLAlchemy as ORM.

##### Create model 

lets call each record a Note. It has  following fields

- text - highlighted text from web page 
- source - url of the web page 
- tags - tags which provide additional details
- comments - in case you want to add something
- createdon/modifiedon - audit fields

    """
    all models 
    """
    import datetime
    from sqlalchemy import Column, Integer, String
    from database import Base
    
    
    
    class Note(Base):
        """
        represent a note record
        """
        __tablename__ = 'note'
        id = Column(Integer, primary_key=True,)
        user_id = Column(String(50), nullable=False)
        text = Column(String(500), nullable=False)
        source = Column(String(200), nullable=False)
        tags = Column(String(500), default='')
        comments = Column(String(500), default='')
        created_on = Column(String(20), default=datetime.datetime.utcnow())
        modified_on = Column(String(20), default=datetime.datetime.utcnow())
    
        def __init__(self, user_id, text, source):
            self.user_id = user_id
            self.text = text
            self.source = source
    
        def serialize(self):

models.py
##### Setup database

To set up database, we are providing the db file path as db location and a function to create all tables as per the model.

    from sqlalchemy import create_engine
    from sqlalchemy.orm import scoped_session, sessionmaker
    from sqlalchemy.ext.declarative import declarative_base
    
    DATABASE = "web-note.db"
    
    engine = create_engine('sqlite:///'+DATABASE, convert_unicode=True)
    db_session = scoped_session(sessionmaker(autocommit=False,
                                             autoflush=False,
                                             bind=engine))
    
    Base = declarative_base()
    Base.query = db_session.query_property()
    
    
    def init_db():
        """
        initialize the db
        """
        # import all modules here that might define models so that
        # they will be registered properly on the metadata.  Otherwise
        # you will have to import them first before calling init_db()
        import models
        Base.metadata.create_all(bind=engine)

database.py
##### Create routes

we are creating primarily 3 routes - 

- Get all the notes  - This uses query.all() to return all the notes from db. Note that there are some issue in serializing these records by sqlAlchemy
- Save a note
- Get a note by Id 

    """
    """
    import json
    import flask
    from flask import request, render_template
    from models import Note
    from database import db_session
    
    
    app = flask.Flask(__name__)
    app.config["DEBUG"] = True
    
    
    @app.route('/', methods=['GET'])
    def home():
        """
        Home
        """
        all_notes = [note.serialize() for note in  Note.query.all()]
        return render_template('index.html', notes=all_notes) 
    
    
    @app.route('/notes', methods=['POST'])
    def notes():
        """
        handler for /notes
        """
        data = request.data.decode('UTF-8')
        data = json.loads(data)
        if data:
            _save_note(data)
        return {'status': 'success'}               
    
    def _save_note(data):
        note = Note(user_id=data.get('user_id'), text=data.get(
                    'text'), source=data.get('source'))
        db_session.add(note)
        db_session.commit()

note_app.py
This is a very basic setup to get what we want. To save the note, we use SQLAlchemy's db_session. To get the list of notes from DB, we use query.all() and render it using jinja.

We need to call the *init_db* from database.py in order to create all the tables as per the models and then we are set to go.

    >>from database import init_db
    >>init_db()
    >>python note_app.py

Code for this is available [here](https://github.com/kapilgarg/web-note/tree/master/web_api/src). 

All the set up is running locally (on 127.0.0.1). Make the necessary changes if you need to deploy this on any server and access it .

---
