import datetime
import pandas as pd
import requests

client_id = '...'
client_secret = '...'
api_host = 'https://stepik.org'
course_id = '170777'

auth = requests.auth.HTTPBasicAuth(client_id, client_secret)
resp = requests.post('https://stepik.org/oauth2/token/',
                         data={'grant_type': 'client_credentials'},
                         auth=auth)
token = resp.json().get('access_token', None)
if not token:
    print('Unable to authorize with provided credentials')
    exit(1)


def fetch_object(obj_class, obj_id):
    api_url = '{}/api/{}s/{}'.format(api_host, obj_class, obj_id)
    response = requests.get(api_url,
                            headers={'Authorization': 'Bearer ' + token}).json()
    return response['{}s'.format(obj_class)][0]


def fetch_objects(obj_class, obj_ids, keep_order=True):
    objs = []
    step_size = 30
    for i in range(0, len(obj_ids), step_size):
        obj_ids_slice = obj_ids[i:i + step_size]
        api_url = '{}/api/{}s?{}'.format(api_host, obj_class,
                                         '&'.join('ids[]={}'.format(obj_id)
                                                  for obj_id in obj_ids_slice))
        response = requests.get(api_url,
                                headers={'Authorization': 'Bearer ' + token}
                                ).json()

        objs += response['{}s'.format(obj_class)]
    if keep_order:
        return sorted(objs, key=lambda x: obj_ids.index(x['id']))
    return objs


course = fetch_object('course', course_id)
sections = fetch_objects('section', course['sections'])
unit_ids = [unit for section in sections for unit in section['units']]
units = fetch_objects('unit', unit_ids)
lesson_ids = [unit['lesson'] for unit in units]
lessons = fetch_objects('lesson', lesson_ids)

moduls = {section['id']: section['title'] for section in sections}
# print(*lessons[4].items(), sep='\n')
# print([moduls[lesson["units"]] for lesson in lessons])

data = {
    "title": [lesson["title"] for lesson in lessons],
    "epic_count": [int(lesson["epic_count"]) for lesson in lessons],
    "abuse_count": [int(lesson["abuse_count"]) for lesson in lessons],
    'date': datetime.date.today(),
}


df = pd.DataFrame(data)
filename = f"lessons_{datetime.date.today()}.xlsx"
df.to_excel(filename, index=False)
