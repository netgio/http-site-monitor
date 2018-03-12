from __future__ import print_function

import os
from datetime import datetime
from urllib2 import urlopen, Request
import json
from datetime import datetime
from datetime import timedelta



SITE = os.environ['site']  # URL of the site to check, stored in the site environment variable, e.g. https://aws.amazon.com
SITE_DOWNLOAD = os.environ['sitedownload']  # URL of the site to check, stored in the site environment variable, e.g. https://aws.amazon.com
EXPECTED = os.environ['expected']  # String expected to be on the page, stored in the expected environment variable, e.g. Amazon
IFTTT_LOG = os.environ['iftttlog'] # IFTTT WebHook URL for successful pings
IFTTT_FAILED = os.environ['iftttfailed'] # IFTTT WebHook URL for failed pings

def validate(res):
    '''Return False to trigger the canary

    Currently this simply checks whether the EXPECTED string is present.
    However, you could modify this to perform any number of arbitrary
    checks on the contents of SITE.
    '''
    return EXPECTED in res
    
def downloadTest():
    ## "downloading with urllib2"
    start_time = datetime.now()
    print('Download started at ' + str(start_time))
    f = urlopen(SITE_DOWNLOAD, timeout=4)
    print('Download opened')
    data = f.read()
    # returns the elapsed milliseconds since the start 
    dt = datetime.now() - start_time
    ms = (dt.days * 24 * 60 * 60 + dt.seconds) * 1000 + dt.microseconds / 1000.0
    print('Download took ' + str(ms))
    return ms

def lambda_handler(event, context):
    print('Checking {} at {}...'.format(SITE, event['time']))
    result = 'passed'
    result_d = 0
    try:
        if validate(urlopen(SITE, timeout=5).read()):
            print('Connection complete')
            result_d = downloadTest()
            print('Downloaded ' + SITE_DOWNLOAD + ' in ' + str(result_d))
        else:
            raise Exception('Validation failed')
    except:
        print('Check failed!')
        r = Request(IFTTT_FAILED)
        r.add_header('Content-type', 'application/json')
        response = urlopen(r, json.dumps({'value1':event['time'],'value2':'failed','value3': result_d}))
        print('Failure Notified')
    else:
        print('Check passed!')
        r = Request(IFTTT_LOG)
        r.add_header('Content-type', 'application/json')
        response = urlopen(r, json.dumps({'value1':event['time'],'value2':result,'value3':result_d}))
        print('Success Notified')
    
    print('Check complete at {}'.format(str(datetime.now())))
    return event['time']
       
