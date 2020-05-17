# The Tech Academy Live Project

## Introduction
For the last 2 weeks of the tech academy we were tasked with creating an application that would be used to manage one’s collection of things related to a hobby. I made an application that one could use to store ratings and reviews of movies they have seen and allow them to organize it by director, actor, or genre. Below is a video of the application in action:

## Code Highlights:
### Back End
* [API](#api)
* [Data scraping](#data-scraping)
* [Database refined search](#database-refined-search)

#### API
I was tasked with connecting to an API, getting a JSON response, parsing through the file, displaying, and then saving chosen information. I created a form that a user would use to search a movie title, which then would be converted into a format that could be passed to the API.

    if request.method == 'POST':
      title = request.POST['title']
      lower_title = title.lower()
      search_title = lower_title.replace(" ", "+")
      api = "http://www.omdbapi.com/"
      parameters = {'apikey': key, 's': title}  # Passes the key and title
      response = requests.get(api, parameters)  # Get's request from API
      movie_data = response.json()
    
If the request was denied I re-rendered the page, passing information to load the modal that is described [later](#modal-on-error).

    if movie_data['Response'] == "False":
      form = FilmSearchForm()  # Render the film search form
      flag = False    # Used in template to bring up modal on response error
      content = {'form': form, 'flag': flag}
      return render(request, 'SceneBetter/scenebetter_search.html', content)

If there were multiple results, I would move the user to a results page to choose the specific film they are looking for, otherwise I would pass film information into a form for the user to write their rating and review and save it in their database.

        if movie_data[dict_key] != "1":
            return redirect('results', title=search_title)
        # Otherwise go straight to review page
        else:
            return redirect('review', title=search_title)   # Passes normalized title to review page

#### Data Scraping
In this story I was tasked with using the Python library Beautiful Soup to display information sourced from another website. I used Bootstrap Cards to display news articles from https://www.imdb.com/news/movie. I had to parse through the returned html from the site to separate the article’s title, content, link to the actual article, and date it was written.
        
        def news(request):
            page = requests.get("https://www.imdb.com/news/movie")  # Get page as an HTML document
            print(page.status_code)
            soup = BeautifulSoup(page.content, 'html.parser')   # Convert page into a navigable object
            # Separate all h2 tags with class title
            all_title_tags = soup.find_all('h2', class_='news-article__title')
            # Separate all div tags with class content
            all_content_tags = soup.find_all('div', class_='news-article__content')
            # Separate all list tags with class date
            all_date_tags = soup.find_all('li', class_='news-article__date')
            articles = []   # Instantiate articles list
            # Create a loop to scrape only 6 articles
            for i in range(0, 6):
                title_tag = all_title_tags[i]   # Get one h2 tag from the list of titles
                title = title_tag.find('a').get_text()  # Get title text that is between classless a tags within h2 tags
                link = title_tag.find('a').get('href')  # Get article link that is within classless a tags within h2 tags
                content_tag = all_content_tags[i]   # Get one div tag from the list of article content
                content = content_tag.get_text()    # Get title text from div tag
                date_tag = all_date_tags[i]     # Get one list tag from the list of dates
                date = date_tag.get_text()      # Get date text from the list tag
                # Create article dictionary to pass title, content, and reference link to template
                article = {'title': title, 'content': content, 'link': link, 'date': date}
                articles.append(article)    # Add each article to the list of articles being passed
            content = {'articles': articles}
            return render(request, 'SceneBetter/scenebetter_news.html', content)
            
#### Database refined search
I wanted users to be able to limit the number of film results that are displayed from their database by searching for a particular actor, director, or genre. I created a form that allowed the user to be able to pick one of the three options and then write a search term. Then I would retrieve the information from the form and query the database for movie objects that contained the search word. If the query set was empty, I re-rendered the page, passing information to load the modal that is described [later](#modal-on-error).

        def films(request):
            form = FilmRefineForm(data=request.POST or None)    # Load film search refine form
            flag = True  # Create a flag to pass that brings up the modal if the queryset is empty
            all_movies = Movie.films.all().order_by('-rating')  # Query all films
            # If the form has been submitted retrieve the search term
            if request.method == 'POST':
                search_word = request.POST['search_word']
                # If there is a genre search query all movies containing the search term, ordered by decreasing by rating
                if request.POST['refine'] == 'genre':
                    movies = Movie.films.all().filter(genre__icontains=search_word).order_by('-rating')
                    # If the queryset is empty pass the false flag(to bring up modal) and all movies
                    if not movies:
                        flag = False
                        content = {'movies': all_movies, 'form': form, 'flag': flag}
                        return render(request, 'SceneBetter/scenebetter_yourfilms.html', content)
                    else:
                        content = {'movies': movies, 'form': form, 'flag': flag}
                        return render(request, 'SceneBetter/scenebetter_yourfilms.html', content)

### Front End
* [Modal on Error](#modal-on-error)
* [Clickable table row](#clickable-table-row)

#### Modal on Error
When the user performs a search to add a film to their database or to refine the films shown from their database, there is a chance their search ends up having no results. For this situation I created a modal, using bootstrap and custom jQuery, that would pop up. I initially had to pass a flag that would change the status of a hidden checkbox when rendering the page dependent on the user’s search.

        <!-- Hidden check box that helps bring up modal if there is a response error from the API-->
        <div class="none">
                <form>
                    <input id="check" type="checkbox" {% if flag == False %} checked  {% endif %}>
                </form>
        </div>
        
Then using jQuery, I would show the modal if the checkbox was checked upon loading the document.

        //Once the page has loaded adds an event handler on the body. When an input with custom attribute check is
        // checked on, the modal is shown
        $(document).ready(function() {
            if ($('#check').is(":checked")) {
                $("#myModal").show()
            }
        });

#### Clickable table row
The user’s stored films are displayed in a table. I wanted the user to be able to click on a table row to then take them to a specific film’s details page. I accomplished this by using a custom data attribute storing the link to the specific page.

        <!--Links to specific detail page by passing pk using a custom attribute for jquery reasons-->
        <tr data-href="{% url 'details' pk=movie.pk %}">
        
Then using jQuery, I put an event handler on the body that when a row with my custom data attribute is clicked, current window link is changed.

        //Once the page has loaded adds an event handler on the body. When a table row with custom attribute data-href is
        // clicked on, the handler changes the current window link to the link in the custom attribute

        $(document).ready(function () {
            $(document.body).on("click", "tr[data-href]", function() {
                window.location.href = this.dataset.href;
            });
        });

