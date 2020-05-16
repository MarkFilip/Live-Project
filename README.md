# The Tech Academy Live Project

## Introduction
For the last 2 weeks of the tech academy we were tasked with creating an application that would be used to manage one’s collection of things related to a hobby. I made an application that one could use to store ratings and reviews of movies they have seen and allow them to organize it by director, actor, or genre. Below is a video of the application in action:

## Code Highlights:
### Back End
•	API
•	Data scraping
•	Database refined search

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
    
If the request was denied I re-rendered the page, passing information to load the modal that was previously described.

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

