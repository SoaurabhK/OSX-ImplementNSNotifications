# OSX-ImplementNSNotifications
NSNotification Implementation Using Blocks

Blocks are normal objects so you can store them in NSArray/NSDictionary.
```
#import <Foundation/Foundation.h>

/** LBNotificationCenter.h */

typedef void (^Observer)(NSString *name, id data);

@interface LBNotificationCenter : NSObject

- (void)addObserverForName:(NSString *)name block:(Observer)block;
- (void)removeObserverForName:(NSString *)name block:(Observer)block;
- (void)postNotification:(NSString *)name data:(id)data;

@end

/** LBNotificationCenter.m */

@interface LBNotificationCenter ()
@property (strong, nonatomic) NSMutableDictionary <id, NSMutableArray <Observer> *> *observers;
@end

@implementation LBNotificationCenter

- (instancetype)init
{
    self = [super init];
    if (self) {
        _observers = [NSMutableDictionary new];
    }
    return self;
}

- (void)addObserverForName:(NSString *)name block:(Observer)block
{
    // check name and block for presence...

    NSMutableArray *nameObservers = self.observers[name];

    if (nameObservers == nil) {
        nameObservers = (self.observers[name] = [NSMutableArray new]);
    }

    [nameObservers addObject:block];
}

- (void)removeObserverForName:(NSString *)name block:(Observer)block
{
    // check name and block for presence...

    NSMutableArray *nameObservers = self.observers[name];

    // Some people might argue that this check is not needed
    // as Objective-C allows messaging nil
    // I prefer to keep it explicit
    if (nameObservers == nil) {
        return;
    }

    [nameObservers removeObject:block];
}

- (void)postNotification:(NSString *)name data:(id)data
{
    // check name and data for presence...

    NSMutableArray *nameObservers = self.observers[name];

    if (nameObservers == nil) {
        return;
    }

    for (Observer observer in nameObservers) {
        observer(name, data);
    }
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString *const Notification1 = @"Notification1";
        NSString *const Notification2 = @"Notification2";

        LBNotificationCenter *notificationCenter = [LBNotificationCenter new];

        Observer observer1 = ^(NSString *name, id data) {
            NSLog(@"Observer1 is called for name: %@ with some data: %@", name, data);
        };
        Observer observer2 = ^(NSString *name, id data) {
            NSLog(@"Observer2 is called for name: %@ with some data: %@", name, data);
        };

        [notificationCenter addObserverForName:Notification1 block:observer1];
        [notificationCenter addObserverForName:Notification2 block:observer2];

        [notificationCenter postNotification:Notification1 data:@"Some data"];
        [notificationCenter postNotification:Notification2 data:@"Some data"];

        [notificationCenter removeObserverForName:Notification1 block:observer1];

        // no observer is listening at this point so no logs for Notification1...
        [notificationCenter postNotification:Notification1 data:@"Some data"];

        [notificationCenter postNotification:Notification2 data:@"Some data"];
    }

    return 0;
}
```

Reference -
http://stackoverflow.com/questions/37308783/how-to-implement-a-nsnotificationcenter-block-style

https://www.mikeash.com/pyblog/friday-qa-2011-07-08-lets-build-nsnotificationcenter.html

https://github.com/mikeash/MANotificationCenter
