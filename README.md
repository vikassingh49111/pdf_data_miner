import PyPDF2
import pdfquery
import json
from pdfminer.pdfdocument import PDFDocument
from pdfminer.pdfpage import PDFPage
from pdfminer.pdfparser import PDFParser
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.converter import PDFPageAggregator
from pdfminer.layout import LAParams, LTTextBox, LTTextLine, LTFigure, LTLine
from PyPDF2 import PdfFileReader
import sys
reload(sys)
sys.setdefaultencoding('utf8')
import pandas as pd
import csv
import numpy as np
from os import path
from datetime import datetime
import math
from scipy.spatial.distance import cdist

def parse_layout(layout):
    
    
    for lt_obj in layout:
        df = pd.DataFrame()
        if isinstance(lt_obj, LTTextLine):
            
            try:
                if (lt_obj.get_text()) is not None:
                    df['text'] = [(lt_obj.get_text())]
                else:
                    df['text'] = None
            except Exception, e:
                df['text'] = None
           
        elif isinstance(lt_obj, LTTextBox):
            parse_layout(lt_obj)
        elif isinstance(lt_obj, LTTextLine):
            parse_layout(lt_obj)
        elif isinstance(lt_obj, LTFigure):
            parse_layout(lt_obj)
            
        try:
            if (lt_obj.__class__.__name__) is not None:
                df['coordinates'] = [(lt_obj.__class__.__name__)]
            else:
                df['coordinates'] = None
        except Exception, e:
            df['coordinates'] = None
            
        try:
            if (lt_obj.bbox) is not None:
                df['bbox'] = [(lt_obj.bbox)]
            else:
                df['bbox'] = None
        except Exception, e:
            df['bbox'] = None
        
        with open("F:/pdftocsv.csv", 'a') as f:
            df.to_csv(f,header=False, index= False)
                            
fp = open('F:/Knote Binder.pdf', 'rb')
parser = PDFParser(fp)
doc = PDFDocument(parser)

rsrcmgr = PDFResourceManager()
laparams = LAParams()
device = PDFPageAggregator(rsrcmgr, laparams=laparams)
interpreter = PDFPageInterpreter(rsrcmgr, device)
for page in PDFPage.create_pages(doc):
    interpreter.process_page(page)
    layout = device.get_result()
    parse_layout(layout)
    #fp.close()
    
    
pdf_info = ['type', 'x0','text']
pdf_data = pd.read_csv('F:/pdftocsv.csv', names=pdf_info)
pdf_data = pd.DataFrame(pdf_data)

pdf_data['type'] = pdf_data['type'].str.rstrip('\n')
pdf_data['type'].replace(' ', np.nan, inplace=True)
#trimAllColumns(pdf_data)
pdf_data1 = pdf_data.dropna(subset  = ['type'])
pdf_data1 = pdf_data1.dropna(subset  = ['text'])

def trimAllColumns(pdf_data1):
    """
    Trim whitespace from ends of each value across all series in dataframe
    """
    trimStrings = lambda x: x.strip() if type(x) is str else x
    return pdf_data1.applymap(trimStrings)
    
csv1=pd.DataFrame()
csv1['text'] = pdf_data1['type'].str.rstrip('\n')
csv1['bbox'] = pdf_data1['text'].str.rstrip(')').str.lstrip('(')
csv1['y1']= csv1['bbox'].apply(lambda x: x[x.rfind(',')+1:])
csv1['x']= csv1['bbox'].apply(lambda x: x[0:x.rfind(',')])
csv1['x1']= csv1['x'].apply(lambda x: x[x.rfind(',')+1:])
csv1['y']= csv1['x'].apply(lambda x: x[0:x.rfind(',')])
csv1['x0']= csv1['y'].apply(lambda x: x[0:x.rfind(',')])
csv1['y0']= csv1['y'].apply(lambda x: x[x.rfind(',')+1:])
csv1 = csv1[['text','x0','y0','x1','y1']]
csv1

csv1['x0'] = csv1['x0'].apply(pd.to_numeric, errors='coerce').astype(int)
csv1['y0'] = csv1['y0'].apply(pd.to_numeric, errors='coerce').astype(int)
csv1['x1'] = csv1['x1'].apply(pd.to_numeric, errors='coerce').astype(int)
csv1['y1'] = csv1['x1'].apply(pd.to_numeric, errors='coerce').astype(int)

a = csv1.loc[csv1['text'] == 'CURRENCY:']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text


a = csv1.loc[csv1['text'] == 'POLICY PERIOD:']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text


a = csv1.loc[csv1['text'] == 'POLICY FORM:']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'POLICY NUMBER:']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'Executive/Insured Entity Wrongdoing']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'Wrongful Employment Practices']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'UNILATERAL DISCOVERY PERCENTAGE']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'BILATERAL DISCOVERY PERCENTAGE']
b = float(a.y0)
d = csv1.loc[csv1['y0'] == b]
d.text

a = csv1.loc[csv1['text'] == 'PAYMENT TERMS:']
b = float(a.x0)
d = csv1.loc[csv1['x0'] == b]
d.text

#takeClosest = lambda num,collection:min(collection,key=lambda x:abs(x-num))
#takeClosest(c,d)

