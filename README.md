MediaCloud Python API Client
============================

This is the source code of the python client for the [MediaCloud API v2](https://github.com/berkmancenter/mediacloud/blob/master/doc/api_2_0_spec/api_2_0_spec.md).

Usage
-----

First [sign up for an API key](https://core.mediacloud.org/login/register).  Then
```
pip install mediacloud
```

Examples
--------

To get the first 2000 the stories associated with a query and dump the output to json:
```python
import mediacloud, json, datetime
mc = mediacloud.api.MediaCloud('MY_API_KEY')

fetch_size = 1000
stories = []
last_processed_stories_id = 0
while len(stories) < 2000:
    fetched_stories = mc.storyList('( obama AND policy ) OR ( whitehouse AND policy)', 
                                   solr_filter=[ mc.publish_date_query( datetime.date(2013,1,1), datetime.date(2015,1,1)), 
                                                                         'tags_id_media:1'],
                                    last_processed_stories_id=last_processed_stories_id, rows= fetch_size)
    stories.extend( fetched_stories)
    if len( fetched_stories) < fetch_size:
        break
    
    last_processed_stories_id = stories[-1]['processed_stories_id']
    
print json.dumps(stories)
```

Find out how many sentences in the US mainstream media that mentioned "Zimbabwe" and "president" in 2013:
```python
import mediacloud, datetime
mc = mediacloud.api.MediaCloud('MY_API_KEY')
res = mc.sentenceCount('( zimbabwe AND president)', solr_filter=[mc.publish_date_query( datetime.date( 2013, 1, 1), datetime.date( 2014, 1, 1) ), 'tags_id_media:1' ])
print res['count'] # prints the number of sentences found
```

Alternatively, this query could be specified as follows
```python
import mediacloud
mc = mediacloud.api.MediaCloud('MY_API_KEY')
results = mc.sentenceCount('( zimbabwe AND president)', '+publish_date:[2013-01-01T00:00:00Z TO 2014-01-01T00:00:00Z} AND +tags_id_media:1')
print results['count']
```

Find the most commonly used words in sentences from the US mainstream media that mentioned "Zimbabwe" and "president" in 2013:
```python
import mediacloud, datetime
mc = mediacloud.api.MediaCloud('MY_API_KEY')
words = mc.wordCount('( zimbabwe AND president)',  solr_filter=[mc.publish_date_query( datetime.date( 2013, 1, 1), datetime.date( 2014, 1, 1) ), 'tags_id_media:1' ] )
print words[0]  #prints the most common word
```

To find out all the details about one particular story by id:
```python
import mediacloud
mc = mediacloud.api.MediaCloud('MY_API_KEY')
story = mc.story(169440976)
print story['url']  # prints the url the story came from
```

To save the first 100 stories from one day to a database:
```python
import mediacloud, datetime
mc = mediacloud.api.MediaCloud('MY_API_KEY')
db = mediacloud.storage.MongoStoryDatabase('one_day')
stories = mc.storyList(mc.publish_date_query( datetime.date (2014, 01, 01), datetime.date(2014,01,02) ), last_processed_stories_id=0,rows=100)
[db.addStory(s) for s in stories]
print db.storyCount()
```

Take a look at the `apitest.py` and `storagetest.py` for more detailed examples.

Development
-----------

If you are interested in adding code to this module, first clone [the GitHub repository](https://github.com/c4fcm/MediaCloud-API-Client).

## Testing

First run all the tests.  Copy `mc-client.config.template` to `mc-client.config` and edit it.
Then run `python tests.py`. Notice you get a `mediacloud-api.log` that tells you about each query it runs.

## Distribution

1. Run `python test.py` to make sure all the test pass
2. Update the version number in `mediacloud/__init__.py`
3. Make a brief note in the version history section in the README file about the changes
4. Run `python setup.py sdist` to test out a version locally
5. Then run `python setup.py sdist upload -r pypitest` to release a test version to PyPI's test server
6. Run `pip install -i https://testpypi.python.org/pypi mediacloud` somewhere and then use it with Python to make sure the test release works.
7. When you're ready to push to pypi run `python setup.py sdist upload -r pypi`
8. Run `pip install mediacloud` somewhere and then try it to make sure it worked.
