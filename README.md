# Major-Project-Report-GEOM2159---Drought-Analyser
Created by s3603187, this repository was created as apart of the Major Project Report for GIS Programming (GEOM2159).  The major project required students to write a plan and program using python code, and as shown in the repository, I have chosen to program a drought analyzing program. 
# -*- coding: utf-8 -*-
"""
Created on Sat Sep 18 13:29:54 2021

@author: Andrew
"""

import os
import processing
from PyQt5 import QtCore, QtGui, QtWidgets

    filepath = 'C:/Users/magli/Desktop/GIS Programming/Project/Python_Files/'
    netCDF_raster = 'sm_pct_Actual_month.nc'
    vectorLga = 'LGA_2020_AUST.shp'
    coordRefSystem = 'EPSG:4283'
    rasterProject = 'Reprojection.tif'
    cellStatsLayer = 'OutputLayer.tif'
    zonalRaster = 'ZonalStats.gpkg'

    ###Step 1: Add all layers
    ###Adds VECTOR layer to QGIS
    lga_layer = iface.addVectorLayer((filepath + vectorLga), vectorLga[:-4], "ogr")
    ###Adds RASTER layer to QGIS
    rastLayer = iface.addRasterLayer((filepath + netCDF_raster), netCDF_raster[:-3])
    ###Dict for reprojection algorithm
    reprojDict = {'DATA_TYPE' : 0,'EXTRA' : '','INPUT' : filepath + netCDF_raster, 'MULTITHREADING' : False, 'NODATA' : None, 'OPTIONS' : '', 'OUTPUT' : filepath + rasterProject, 'RESAMPLING' : 0, 'SOURCE_CRS' : None, 'TARGET_CRS' : QgsCoordinateReferenceSystem('EPSG:4283'), 'TARGET_EXTENT' : None, 'TARGET_EXTENT_CRS' : None, 'TARGET_RESOLUTION' : None }
    ###Output location to save file of reprojection
    {'OUTPUT': filepath + rasterProject}
    ###Runs process to create algorithm
    processing.run("gdal:warpreproject", reprojDict)
    ###Adds reprojection file
    projLayer = iface.addRasterLayer((filepath + rasterProject), rasterProject[:-4])

    ###Step 2: Cell Statistics
    ###Dictionary for cell statistic parameters
    cellStatsParam = { 'IGNORE_NODATA' : True, 'INPUT' : [filepath + rasterProject], 'OUTPUT' : filepath + cellStatsLayer, 'OUTPUT_NODATA_VALUE' : -9999, 'REFERENCE_LAYER' : filepath + rasterProject, 'STATISTIC' : 2 }
    ###runs algorithm process to create cell stats layer
    processing.run("qgis:cellstatistics", cellStatsParam)
    ###add layer to QGIS to debug
    cellStatsLayer = iface.addRasterLayer((filepath + cellStatsLayer), netCDF_raster[:-4])

    ###Step 3: Add field to Vector file
    ###Begin editting the LGA layer
    lga_layer.startEditing()
    ###Field to be added is "SM_M"
    lga_layer.dataProvider().addAttributes([QgsField("SM_M", QVariant.String)])
    ###Update and commit changes to apply additions
    lga_layer.updateFields()
    lga_layer.commitChanges()

    ###Step 4: Zonal Statistics
    ###Zonal dictionary for algorithm created
    zonalDict = { 'COLUMN_PREFIX' : '_', 'INPUT' : filepath + vectorLga, 'INPUT_RASTER' : 'C:/Users/magli/Desktop/GIS Programming/Project/Python_Files/OutputLayer.tif', 'OUTPUT' : filepath + zonalRaster, 'RASTER_BAND' : 1, 'STATISTICS' : [0,1,2] }
    ###Output to save zonal file in folder
    {'OUTPUT': filepath + zonalRaster}
    ###Run algorithm of zonal statistics
    processing.run("qgis:zonalstatisticsfb", zonalDict)
    ###Add zonal stats vector layer
    zonalVector = iface.addVectorLayer((filepath + zonalRaster), vectorLga[:-4], "ogr")

    ###Step 5: Input the classification of SM_M 
    ###getFeature to obtain all values within the layer
    dataSet = zonalVector.getFeatures()
    ###"for data" goes through data
    for data in dataSet:
    ###variable used to specifiy field within the layer
        status = data["_mean"]
    ###if statement to clasify each vulnerability conditions within variable field  
    ###when if statement = true, string variable applies
        if status > 0.00 and status <= 0.100:
            condition = "Extremely Below Average"
        elif status > 0.100 and status <= 0.200:
            condition = "Very Much Below Average"
        elif status > 0.200 and status <= 0.400:
            condition = "Below Average"
        elif status > 0.400 and status <= 0.600:
            condition = "Average"
        elif status > 0.600 and status <= 0.800:
            condition = "Above Average"
        elif status > 0.800 and status <= 0.900:
            condition = "Very Much Above Average"
        elif status > 0.900 and status <= 1.000:
            condition = "Extremely Above Average"
        else:
            condition = "N/A"
    ###Edit layer to make changes
        zonalVector.startEditing()
    ###where if statement true SM_M field = condition string ^^^
        data["SM_M"] = condition
    ###update and commit changes to keep values in field
        zonalVector.updateFeature(data)
        zonalVector.commitChanges()

    ###Step 6: Creating Gradient Symbology 
    ###Obtain all polygons and create if loop for SM_M field
    dataSet = zonalVector.getFeatures()
    ###For statement to symbolize polygons
    for data in dataSet:
        ###List used to collect arguments for QgsRendererRange
        myRangeList = []
        myOpacity = 1
        ###Retrieves field from attributes table within layer
        myTargetField = "_mean" 

        ###First label
        ###Min and Max for data values within "myTargetField"
        myMin = 0.0000
        myMax = 0.1000
        myLabel = 'Group 1'
        ###myColour will symbolize the values within min and max
        myColour = QtGui.QColor('#ca0020')
        ###Stores geometry layer into variable
        mySymbol1 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        ###Changes colour of geometry variable and opacity
        mySymbol1.setColor(myColour)
        mySymbol1.setOpacity(myOpacity)
        ###Arguments for symbology layer
        myRange1 = QgsRendererRange(myMin, myMax, mySymbol1, myLabel)
        ###Adjusts list of myRangeList to input variable with arguments
        myRangeList.append(myRange1)

        ###Second label
        myMin = 0.1001
        myMax = 0.2000
        myLabel = 'Group 2'
        myColour = QtGui.QColor('#e66e61')
        mySymbol2 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol2.setColor(myColour)
        mySymbol2.setOpacity(myOpacity)
        myRange2 = QgsRendererRange(myMin, myMax, mySymbol2, myLabel)
        myRangeList.append(myRange2)

        ###Third label
        myMin = 0.2001
        myMax = 0.4000
        myLabel = 'Group 3'
        myColour = QtGui.QColor('#f5c0a9')
        mySymbol3 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol3.setColor(myColour)
        mySymbol3.setOpacity(myOpacity)
        myRange3 = QgsRendererRange(myMin, myMax, mySymbol3, myLabel)
        myRangeList.append(myRange3)

        ###Fourth label
        myMin = 0.4001
        myMax = 0.6000
        myLabel = 'Group 4'
        myColour = QtGui.QColor('#f7f7f7')
        mySymbol4 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol4.setColor(myColour)
        mySymbol4.setOpacity(myOpacity)
        myRange4 = QgsRendererRange(myMin, myMax, mySymbol4, myLabel)
        myRangeList.append(myRange4)

        ###Fifth label
        myMin = 0.6001
        myMax = 0.8000
        myLabel = 'Group 5'
        myColour = QtGui.QColor('#b4d6e6')
        mySymbol5 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol5.setColor(myColour)
        mySymbol5.setOpacity(myOpacity)
        myRange5 = QgsRendererRange(myMin, myMax, mySymbol5, myLabel)
        myRangeList.append(myRange5)

        ###Sixth label
        myMin = 0.8001
        myMax = 0.9000
        myLabel = 'Group 6'
        myColour = QtGui.QColor('#63a9cf')
        mySymbol6 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol6.setColor(myColour)
        mySymbol6.setOpacity(myOpacity)
        myRange6 = QgsRendererRange(myMin, myMax, mySymbol6, myLabel)
        myRangeList.append(myRange6)

        ###Final label
        myMin = 0.9001
        myMax = 1.0
        myLabel = 'Group 7'
        myColour = QtGui.QColor('#0571b0')
        mySymbol7 = QgsSymbol.defaultSymbol(zonalVector.geometryType())
        mySymbol7.setColor(myColour)
        mySymbol7.setOpacity(myOpacity)
        myRange7 = QgsRendererRange(myMin, myMax, mySymbol7, myLabel)
        myRangeList.append(myRange7)

        myRenderer = QgsGraduatedSymbolRenderer('', myRangeList)
        myClassificationMethod = QgsApplication.classificationMethodRegistry().method("PrettyBreaks")
        myRenderer.setClassificationMethod(myClassificationMethod)
        myRenderer.setClassAttribute(myTargetField)
        zonalVector.setRenderer(myRenderer)
        QgsProject.instance().addMapLayer(zonalVector)

