![](http://www.ciscolive.com/global/wp-content/uploads/cisco-logo-blue.png)


**Getting Started**

 Download the sample   [link]()
  
##Installing Dependencies
We use cocoapods to manage dependencies [https://cocoapods.org/](Cocoapods)

- Run the command **sudo gem install cocoapods** (if you do not already have cocoapods installed)
- cd to the directory where your project directory is
- run the command **pod install**
- open **SampleApp.xcworkspace** (not the .xcodeproj file)
-  Pods for the sdk is WIP
 
##  In the sample iOS application

- Download the framework from [link]() and 	drag and drop .framework  file to demo project.
	
	<!--![image](http://gdurl.com/cwMA)-->
	     
- Add Organization through Qrcode  located under Mobile - [xy61il.cmxcis.co](https://xy61il.cmxcis.co)



##Check out the below snippets that demonstrate some  common scenarios.


**Connecting to Device**

```objective-c
    [[MistManager sharedInstance] addEvent:@"didConnect" forTarget:self];

	@params  isConnected represents if user are successfully connected to device*
	-(void)mistManager:(MSTCentralManager *)manager didConnect:(BOOL)isConnected  
```

**Check if you are inside tracking zone** 

    [[MistManager sharedInstance] addEvent:@"didUpdateMap" forTarget:self];
    -(void)mistManager:(MSTCentralManager *)manager didUpdateMap:(MSTMap *)map at:(NSDate *)dateUpdated{
    [Default performBlockOnMainThread:^{
        if (map) {
        // You are Indoor
        [self.view makeToast:[NSString stringWithFormat:@"You are in %@",map.mapName]];
  
    } else {
       // You are out of tracking Zone  or Outdoor
        [self.view makeToast:@"You are OutDoor"];
    }
    }];}

    

  **Get Floors  and Bluedot**
  
    [[MistManager sharedInstance] addEvent:@"didUpdateRelativeLocation" forTarget:self];   
    
    
 This function is called when we have both relative location and the actual map details for rendering.
  Called when Framework receives a location update  and has map information.
 

 	
 	-(void)mistManager: (MSTCentralManager *) manager didUpdateRelativeLocation: (MSTPoint *) relativeLocation inMaps: (NSArray *) maps at: (NSDate *) dateUpdated{
 	To draw the floorimage in a UIView initialze the MSTWayfinder and pass the Map object to  MSTWayfinder
	For more info please see the the sample app for how to add a floor image 
	To draw a blue dot use the below method of MSTWayfinder.
	Check out the sample app code for details 
	drawDotViewAtCGPoint:(CGPoint)point forIndex:(NSUInteger)index shouldMove:(BOOL)shouldMove shouldShowMotion:(bool)showMotion
 	}
 	
 	
        
        

**Get Zone and virtual beacon notification**

    [[MistManager sharedInstance] addEvent:@"didReceiveNotificationMessage" forTarget:self];   
    
    -(void)mistManager:(MSTCentralManager *)manager didReceiveNotificationMessage:(NSDictionary *)payload{

        NSDictionary *message = [payload objectForKey:@"message"];
        if ([[payload objectForKey:@"type"] isEqualToString:@"zones-events"]) {
            [self handleZoneEvents:message];
        }
        if ([[payload objectForKey:@"type"] isEqualToString:@"zone-event-vb"]) {
            [self handleZoneVBEvents:message];
        }
	}

 **To show all path**

MSTMap contains the Wayfinding json which contain all the  nodes with its edges and position 
Below is the sample json

		"nodes": [{
		"position": {
			"y": 1143.010162560576,
			"x": 377.429862471456
		},
		"edges": {
			"A5": "1",
			"B": "1",
			"E": "1",
			"G1": "1",
			"H": "1"
		},
		"name": "G"
	} 
	
	
Below is the sample Snippet to create the wayfinding graph from the Json

        -(void)loadWayfindingData:(NSDictionary *)mapJSON{
        NSArray *nodesFromFile = [mapJSON objectForKey:@"nodes"];

        
        MSTGraph *graph = [[MSTGraph alloc] init];
        NSMutableDictionary *nodes = [[NSMutableDictionary alloc] init];
        
        for (NSDictionary *aNode in nodesFromFile) {
            CGFloat x = [aNode[@"position"][@"x"] doubleValue];
            CGFloat y = [aNode[@"position"][@"y"] doubleValue];
            [self.nodes setObject:[[MSTNode alloc] initWithName:aNode[@"name"] andPoint:CGPointMake(x,y) andEdges:aNode[@"edges"]] forKey:aNode[@"name"]];
        }
        for (NSString *key in self.nodes) {
            MSTNode *node = self.nodes[key];
            [self calculateEdgeDistanceForNode:node];
            [graph addVertex:node.nodeName withEdges:node.edges];
        }

    }	
    
    }

	-(void)calculateEdgeDistanceForNode:(MSTNode *)node{
    NSMutableDictionary *tempEdge = [[NSMutableDictionary alloc] init];
    for (NSString *nodeName in node.edges) {
        MSTNode *otherNode = [self.nodes objectForKey:nodeName];
        double x = pow(node.nodePoint.x-otherNode.nodePoint.x, 2)+pow(node.nodePoint.y-otherNode.nodePoint.y, 2);
        double d = sqrt(x);
        [tempEdge setObject:[NSString stringWithFormat:@"%lf",d] forKey:nodeName];
    }
    node.edges = [tempEdge mutableCopy];
	}

Once you added the MSTWayfinder view in your viewcontroller you can set the boolean showSkeletonView true/false to see all possible path

**To draw Wayfinding from origin to destination**

	MSTWayfinder indoorMapview	 = [[MSTWayfinder alloc] initWithFrame:self.view.bounds];

                                [indoorMapview setOriginNodeUsingPoint:startingFVPoint];
                                
                                dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0L), ^{
                                    CGPathRef path = [self.indoorMapView renderWayfinding2];
                                    if (path != NULL) {
                                        CGPathRetain(path);
                                        [Default performBlockOnMainThread:^{
                                            [indoorMapview drawWayUsingPath:path enable:true];
                                            CGPathRelease(path);
                                        }];
                                    }
                                });
                            
                            
                        






	   


 


    
    
    
    
    
   







