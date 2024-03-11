from IPython.display import display, Image, clear_output, HTML
import time
import ipywidgets as widgets
from IPython.display import display, clear_output, HTML
import json 
from IPython.display import HTML, display

all_images = ["Variation1.jpg","Variation2.jpg","Variation3.jpg","Variation4.jpg","Variation5.jpg",
              "Variation6.jpg","Variation7.jpg","Variation8.jpg"]

image_list = all_images.copy()

random.shuffle(image_list)

print(image_list)

def send_to_google_form(data_dict, form_url):
    ''' Helper function to upload information to a corresponding google form 
        You are not expected to follow the code within this function!
    '''
    form_id = form_url[34:90]
    view_form_url = f'https://docs.google.com/forms/d/e/{form_id}/viewform'
    post_form_url = f'https://docs.google.com/forms/d/e/{form_id}/formResponse'

    page = requests.get(view_form_url)
    content = BeautifulSoup(page.content, "html.parser").find('script', type='text/javascript')
    content = content.text[27:-1]
    result = json.loads(content)[1][1]
    form_dict = {}
    
    loaded_all = True
    for item in result:
        if item[1] not in data_dict:
            print(f"Form item {item[1]} not found. Data not uploaded.")
            loaded_all = False
            return False
        form_dict[f'entry.{item[4][0][0]}'] = data_dict[item[1]]
    
    post_result = requests.post(post_form_url, data=form_dict)
    return post_result.ok

form_url = 'https://docs.google.com/forms/d/e/1FAIpQLSeh-CmaGT4N6TKWmhvyhYMsmgtjHZ15sV7s2o_8_G_d7ouYwg/viewform'

event_info = {
    'type': '',
    'description': '',
    'time': -1
}

def wait_for_event(timeout=-1, interval=0.001, max_rate=20, allow_interupt=True):    
    start_wait = time.time()

    # set event info to be empty
    # as this is dict we can change entries
    # directly without using
    # the global keyword
    event_info['type'] = ""
    event_info['description'] = ""
    event_info['time'] = -1

    n_proc = int(max_rate*interval)+1
    
    with ui_events() as ui_poll:
        keep_looping = True
        while keep_looping==True:
            # process UI events
            ui_poll(n_proc)

            # end loop if we have waited more than the timeout period
            if (timeout != -1) and (time.time() > start_wait + timeout):
                keep_looping = False
                
            # end loop if event has occured
            if allow_interupt==True and event_info['description']!="":
                keep_looping = False
                
            # add pause before looping
            # to check events again
            time.sleep(interval)
    
    # return event description after wait ends
    # will be set to empty string '' if no event occured
    return event_info

# this function lets buttons 
# register events when clicked
def register_event(btn):
    # display button description in output area
    global correct_answers, right_answer
    event_info['type'] = "click"
    event_info['description'] = btn.description
    event_info['time'] = time.time()
    if btn.description.lower() == correct_side:  
        correct_answers += 1
        right_answer = 1
    else:
        right_answer = 0

import random
import pandas as pd

random.seed(1)

print("Welcome to the ANS test")

id_instructions = """

Enter your anonymised ID

To generate an anonymous 4-letter unique user identifier please enter:
- two letters based on the initials (first and last name) of a childhood friend
- two letters based on the initials (first and last name) of a favourite actor / actress

e.g. if your friend was called Charlie Brown and film star was Tom Cruise
     then your unique identifier would be CBTC
"""

print(id_instructions)
ans1 = input("> ")

clear_output(wait=False)

data_consent_info = """DATA CONSENT INFORMATION:

Please read:

We wish to record your response data
to an anonymised public data repository. 
Your data will be used for educational teaching purposes
practising data analysis and visualisation.

Please type   yes   in the box below if you consent to the upload."""

print(data_consent_info)
rslt = input("> ") 

if rslt == "yes": 
    print("Thanks for your participation.")
    print("Please contact a.fedorec@ucl.ac.uk")
    print("If you have any questions or concerns")
    print("regarding the stored results.")
    time.sleep(2)
else: 
    raise(Exception("User did not consent to continue test."))

clear_output(wait=False)

print("Please enter your gender (male,female or other):")
ans2 = input("> ")

print("How many hours of sleep did you get last night?")
ans3 = input(">")

clear_output(wait=False)

instructions = """
Click either left or right for which circle you think has the most dots.
You will be shown the image for 0.75 seconds, have a response time of 3 seconds

There will be 50 questions."""

print(instructions)
time.sleep(10)
clear_output(wait=False) 

print("The test will start in 5 seconds")
time.sleep(5)
clear_output(wait=False)

questions = 50
correct_answers = 0

results_dict = {
    'filename' : [],
    'rsponse_time' : [],
    'ratio' : [],
    'correct' : []
}

"""
btn3 = widgets.Button(description="Start")
btn3.on_click(start_btn) 
start = widgets.HBox([btn3])
display(start)
"""
for i in range(questions):
        
    random_image = random.choice(image_list)
    display(Image(filename=random_image, width =700))
        
    time.sleep(0.75)
    clear_output(wait=False)
    
    start_time = time.time() 

    btn1 = widgets.Button(description="Left")
    btn2 = widgets.Button(description="Right")
    
    btn1.on_click(register_event) 
    btn2.on_click(register_event) 
        
    response = HTML("Which side has the most dots?")
    display(response)
    
    btn1.layout.width = '300px' # Adjust width 
    btn1.layout.height = '100px' # Adjust height 
    
    btn2.layout.width = '300px' # Adjust width 
    btn2.layout.height = '100px' # Adjust height 
        
    panel = widgets.HBox([btn1, btn2])
    display(panel)
    
    if int(random_image[-5]) in [1, 4, 6, 7]:
        correct_side = "right"
    else:
        correct_side = "left"
        
    if int(random_image[-5]) in [1,2,3]: 
        rtio = 0.75
    elif int(random_image[-5]) in [4,5]:
        rtio = 0.857
    elif int(random_image[-5]) in [6]:
        rtio = 0.889
    else:
        rtio = 0.9
        
    result = wait_for_event(timeout=3)
    clear_output()
    
    end_time = time.time()
    response_time =  end_time - start_time
    
    if result['description']!="":
        print(f"User clicked: {result['description']}")
        time.sleep(1.5)
    else:
        print("User did not click in time")
        time.sleep(1.5)
        clear_output(wait=False)
    
        
    results_dict['filename'].append(random_image)
    results_dict['rsponse_time'].append(response_time)
    results_dict['correct'].append(right_answer)
    results_dict['ratio'].append(rtio)

resultsdataframe = pd.DataFrame(results_dict)

clear_output(wait=True) 
print(f"You got {correct_answers} out of {questions} correct.")

data_dict = {
    'name': ans1,
    'gender': ans2,
    'sleep': ans3,
    'score' : correct_answers,
    'result_json' : resultsdataframe.to_json()
}

if rslt == "yes":
    send_to_google_form(data_dict, form_url)
