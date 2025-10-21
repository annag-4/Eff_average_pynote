#!/usr/bin/env python
# coding: utf-8

# In[ ]:


import pandas as pd
import os
import ROOT


# In[ ]:


def parse_efficiency_table(csv_table_path,filter_status = ''):
    # Read the CSV file into a DataFrame
    df = pd.read_csv(csv_table_path)
    if filter_status == '':
        return df
    else:
        return df[df['last_global'] == filter_status]


# In[ ]:


example_table = '../data/grafana_efficiency_table.csv'


# In[ ]:





# In[ ]:


def getPlot(filename,sector,station,hist_pattern):
    """Get the track hit distribution from the DQM file"""
    file = ROOT.TFile(filename)
    arm = 0 if sector == '45' else 1
    hist_path = hist_pattern.format(
        sector=sector,
        station=station,
        arm=arm
    )
    
    if not file:
        print(f'Failed to open file: {filename}')
        file.Close()
        return None
    
    h = file.Get(hist_path)
    if not h:
        print(f'Failed to retrieve histogram: {hist_path}')
        file.Close()
        return None
    
    # Create a copy of the histogram
    hist_class = h.Class().GetName()
    if hist_class == 'TH1D':
        hist = ROOT.TH1D(h)
    elif hist_class == 'TH2D':
        hist = ROOT.TH2D(h)
    else:
        print(f'Found histogram of non-supported class: {hist_class}')
        file.Close()
        return None
        
    # Pass the ownership to the top-level application
    hist.SetDirectory(0)
    
    return hist


# In[ ]:


def merge_efficiency_analysis_output(runs,input_dir,output_file_path):
    checkSum = False
    input_file_pattern = '{run}/outputEfficiencyAnalysisDQMHarvester_run{run}.root'
    if os.path.exists(output_file_path):
        print(output_file_path+' already existed')
        print('Moving it to '+output_file_path+'.old')
        os.rename(output_file_path,output_file_path+'.old')
    if os.path.exists(output_file_path+'.tmp'):
        os.remove(output_file_path+'.tmp')
    
    plotToCheck = 'DQMData/Run 999999/Arm{arm}/Run summary/st{station}/rp3/h2AuxEfficiencyMap_arm{arm}_st{station}_rp3_pl0'
    realSumEntries = 0
    
    # Get all the output files and hadd them
    input_file_paths = []
    for run in runs:
        input_file_path = input_dir+'/'+input_file_pattern.format(run=run)
        input_file_paths.append(input_file_path)
        if checkSum:
            hist = getPlot(input_file_path,'56','2',plotToCheck)
            entries = hist.GetEntries()
            realSumEntries += entries        
    
    # Save to a temporary output, then re-create the efficiency histograms
    hadd_command = 'hadd {output_file} {input_files}'.format(
        output_file = output_file_path+'.tmp',
        input_files = ' '.join(input_file_paths)
    )
    os.system(hadd_command)
    
    if checkSum:
        hist = getPlot(output_file_path+'.tmp','56','2',plotToCheck)
        finalEntries = hist.GetEntries()
        print('The total entries in the summed hist are '+str(finalEntries))
        print('The sum of all entries of the addenda is '+str(realSumEntries))
        
        
    # Re-create efficiency histograms with aggregated num/den
    sectors = ['45','56']
    stations = ['0','2']
    planes = [0,1,2,3,4,5]
    efficiency_num_pattern = 'DQMData/Run 999999/Arm{arm}/Run summary/st{station}/rp3/h2AuxEfficiencyMap_arm{arm}_st{station}_rp3_pl{plane}'
    efficiency_den_pattern = 'DQMData/Run 999999/Arm{arm}/Run summary/st{station}/rp3/h2EfficiencyNormalizationMap_arm{arm}_st{station}_rp3_pl{plane}'
    ofile = ROOT.TFile(output_file_path,'RECREATE')
    for sector in sectors:
        arm = 0 if sector == '45' else 1
        for station in stations:
            ofile.mkdir('DQMData/Run 999999/Arm{arm}/Run summary/st{station}/rp3/'.format(arm=arm,station=station))
            for plane in planes:
                efficiency_num_pattern_plane = efficiency_num_pattern.format(arm='{arm}',station='{station}',plane=plane)
                efficiency_den_pattern_plane = efficiency_den_pattern.format(arm='{arm}',station='{station}',plane=plane)
                hist_num = getPlot(output_file_path+'.tmp',sector,station,efficiency_num_pattern_plane)
                hist_den = getPlot(output_file_path+'.tmp',sector,station,efficiency_den_pattern_plane)
                ofile.cd('DQMData/Run 999999/Arm{arm}/Run summary/st{station}/rp3/'.format(arm=arm,station=station))
                hist_num.Write()
                hist_den.Write()
                hist_eff = hist_num.Clone()
                name='h2EfficiencyMap_arm{arm}_st{station}_rp3_pl{plane}'.format(arm=arm,station=station,plane=plane)
                hist_eff.SetNameTitle(name,name+';x (mm);y (mm)')
                hist_eff.Divide(hist_den)
                hist_eff.Write()
    ofile.Close()
    
    os.remove(output_file_path+'.tmp')


# ## Generate avg efficiency files here

# In[ ]:



# In[ ]:




