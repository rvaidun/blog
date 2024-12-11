---
layout: post
title:  "Reverse Engineering Apple Music iOS App"
date:   2024-03-17 12:52:08 -0800
categories: reverse-engineering
---

# Preface
The goal of this was to document my journey reverse engineering the Apple Music app to make the queue more like Spotify. Pre iOS 18 the Apple Music queue acted more as a stack than a queue and I wanted to modify the queue feature to be more like a queue. In iOS 18 it the Apple Music queue was updated to act more like Spotify so I have abandoned working on it. I decided to publish the post even though it is extremely rough and unfinished to share my findings and hopefully to help others to start reverse engineering with iOS. I also used this blog as a notebook to remember stuff I found earlier when reverse engineering. 
# Setup
Setting up debugserver entitlements

Using debugserver after scping to iphone
```
➜  Payload git:(master) ✗ ssh mobile@192.168.1.29
(mobile@192.168.1.29) Password for mobile@Rahuls-Developer-iPhone:
Rahuls-Developer-iPhone:~ mobile% /var/jb/usr/bin/debugserver 0.0.0.0:4445 -a Music
debugserver-@(#)PROGRAM:LLDB  PROJECT:lldb-1300.2.10
 for arm64.
Attaching to process Music...
error: failed to attach to process named: ""
Exiting.
```

# LLDB

Initial setup
```
(lldb) platform select remote-ios
(lldb) process connect connect://192.168.1.29:4445
```

In Apple Music if you want to queue songs you can swipe right on the song and it will queue. After swiping on some songs to load the proper classes into memory I take a recursiveDescription of the keyWindow. This gives a hierarchy for the UI

```
(lldb) po [[UIWindow keyWindow] recursiveDescription]
...
<UISwipeActionPullView: 0x104495100; cellEdge = UIRectEdgeLeft, actions = <NSArray: 0x282ca7030>>
    | <UISwipeActionStandardButton: 0x1044fdb60; frame = (74 0; 74 48); anchorPoint = (1, 0.5); opaque = NO; autoresize = W+H; tintColor = UIExtendedGrayColorSpace 1 1; layer = <CALayer: 0x28222a880>>
    | <UIView: 0x1044e7470; frame = (-517 0; 591 48); layer = <CALayer: 0x282228660>>
    | <UIImageView: 0x10987fc70; frame = (26 16; 22 16.5); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x28222a4e0>>
    | <UIButtonLabel: 0x109804f20; frame = (27 7.5; 47 33); text = 'Playing Last'; hidden = YES; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x2800de490>>
    | <UISwipeActionStandardButton: 0x1098c4fb0; frame = (0 0; 74 48); anchorPoint = (1, 0.5); opaque = NO; autoresize = W+H; tintColor = UIExtendedGrayColorSpace 1 1; layer = <CALayer: 0x28222a9c0>>
    | <UIView: 0x10982a880; frame = (-517 0; 591 48); layer = <CALayer: 0x28222ae80>>
    | <UIImageView: 0x10982b1f0; frame = (26 15.5; 22 17); clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x282229f60>>
    | <UIButtonLabel: 0x1098c52a0; frame = (27 7.5; 47 33); text = 'Playing Next'; hidden = YES; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x2800dead0>>
	...
```
From the output of this I search for keywords that has to do with queue songs and I found the string `Playing Next`

After some google searching I find the header file for [UISwipeActionPullView](https://developer.limneos.net/index.php?ios=11.0&framework=UIKit.framework&header=UISwipeActionPullView.h). There is no official Apple documentation on this class. We can also see these instace methods with lldb.
```
(lldb) po [@"" __methodDescriptionForClass:[UISwipeActionPullView class]]

in UISwipeActionPullView:
	Properties:
		@property (nonatomic, getter=_roundedStyleCornerRadius, setter=_setRoundedStyleCornerRadius:) double roundedStyleCornerRadius;  (@synthesize roundedStyleCornerRadius = _roundedStyleCornerRadius;)
		@property (weak, nonatomic) <UISwipeActionPullViewDelegate>* delegate;  (@synthesize delegate = _delegate;)
		@property (readonly, nonatomic) unsigned long cellEdge;  (@synthesize cellEdge = _cellEdge;)
		@property (nonatomic) struct UIEdgeInsets contentInsets;  (@synthesize contentInsets = _contentInsets;)
		@property (copy, nonatomic) UIColor* backgroundPullColor;  (@synthesize backgroundPullColor = _backgroundPullColor;)
		@property (readonly, nonatomic) UIContextualAction* primarySwipeAction;
		@property (readonly, nonatomic) double currentOffset;  (@synthesize currentOffset = _currentOffset;)
		@property (readonly, nonatomic) double openThreshold;
		@property (readonly, nonatomic) double confirmationThreshold;
		@property (readonly, nonatomic) UIColor* primaryActionColor;
		@property (readonly, nonatomic) BOOL primaryActionIsDestructive;
		@property (readonly, nonatomic) BOOL hasActions;
		@property (nonatomic) BOOL buttonsUnderlapSwipedView;  (@synthesize buttonsUnderlapSwipedView = _buttonsUnderlapSwipedView;)
		@property (nonatomic) BOOL autosizesButtons;  (@synthesize autosizesButtons = _autosizesButtons;)
		@property (nonatomic) unsigned long state;  (@synthesize state = _state;)
	Instance Methods:
		- (void) freeze; (0x18413260c)
		- (struct UIEdgeInsets) contentInsets; (0x184133c34)
		- (void) setDelegate:(id)arg1; (0x184133c20)
		- (void) moveToOffset:(double)arg1 extraOffset:(double)arg2 animator:(id)arg3 usingSpringWithStiffness:(double)arg4 initialVelocity:(double)arg5; (0x184132958)
		- (void) setContentInsets:(struct UIEdgeInsets)arg1; (0x184133c4c)
		- (double) currentOffset; (0x184133bf0)
		- (unsigned long) cellEdge; (0x184133be0)
		- (void) layoutSubviews; (0x184131ad4)
		- (void) _setWidth:(double)arg1; (0x184131fb0)
		- (void) resetView; (0x184132364)
		- (unsigned long) state; (0x184133cb0)
		- (id) delegate; (0x184133c00)
		- (void) .cxx_destruct; (0x184133cf0)
		- (void) setFrame:(struct CGRect)arg1; (0x184131e90)
		- (BOOL) hasActions; (0x184130e8c)
		- (id) hitTest:(struct CGPoint)arg1 withEvent:(id)arg2; (0x18413382c)
		- (Class) _buttonClass; (0x184131240)
		- (double) openThreshold; (0x184130f80)
		- (double) _roundedStyleCornerRadius; (0x184133cd0)
		- (void) _setRoundedStyleCornerRadius:(double)arg1; (0x184133ce0)
		- (BOOL) autosizesButtons; (0x184133c90)
		- (id) primarySwipeAction; (0x184130dec)
		- (void) setAutosizesButtons:(BOOL)arg1; (0x184133ca0)
		- (void) configureWithSwipeActions:(id)arg1; (0x1841328b4)
		- (double) confirmationThreshold; (0x184131154)
		- (id) sourceViewForAction:(id)arg1; (0x1841334c8)
		- (void) _performAction:(id)arg1 offset:(double)arg2 extraOffset:(double)arg3; (0x184133404)
		- (BOOL) buttonsUnderlapSwipedView; (0x184133c80)
		- (id) initWithFrame:(struct CGRect)arg1 cellEdge:(unsigned long)arg2 style:(unsigned long)arg3; (0x184130cdc)
		- (void) setBackgroundPullColor:(id)arg1; (0x184133c74)
		- (void) setState:(unsigned long)arg1; (0x184133cc0)
		- (void) setButtonsUnderlapSwipedView:(BOOL)arg1; (0x184130f04)
		- (BOOL) primaryActionIsDestructive; (0x184130e40)
		- (unsigned long) _swipeActionCount; (0x184131228)
		- (double) _paddingToSwipedView; (0x184131294)
		- (double) _totalInterButtonPadding; (0x1841312d4)
		- (double) _directionalMultiplier; (0x1841311fc)
		- (double) _interButtonPadding; (0x1841312b4)
		- (void) _setupClippingViewIfNeeded; (0x184131334)
		- (void) _tappedButton:(id)arg1; (0x1841336b0)
		- (void) _pressedButton:(id)arg1; (0x184133788)
		- (void) _unpressedButton:(id)arg1; (0x1841337e0)
		- (void) _rebuildButtons; (0x1841314c0)
		- (void) _layoutClippingLayer; (0x184132244)
		- (void) _setLayerBounds:(struct CGRect)arg1; (0x184132004)
		- (id) primaryActionColor; (0x184130eb0)
		- (id) backgroundPullColor; (0x184133c64)
		- (id) description; (0x184133ad4)
		- (void) setBounds:(struct CGRect)arg1; (0x184131f14)
```
LLDB also shows the address instance functions. `_tappedButton` looks interesting so I set a breakpoint there and take a look at it
```
(lldb) b -a 0x1841336b0
Breakpoint 13: where = UIKitCore`-[UISwipeActionPullView _tappedButton:], address = 0x00000001841336b0
(lldb) con
Process 29939 resuming
Process 29939 stopped // It stopped after I tapped the "Play Next" button
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 13.1
    frame #0: 0x00000001841336b0 UIKitCore`-[UISwipeActionPullView _tappedButton:]
UIKitCore`-[UISwipeActionPullView _tappedButton:]:
->  0x1841336b0 <+0>:  stp    x24, x23, [sp, #-0x40]!
    0x1841336b4 <+4>:  stp    x22, x21, [sp, #0x10]
    0x1841336b8 <+8>:  stp    x20, x19, [sp, #0x20]
    0x1841336bc <+12>: stp    x29, x30, [sp, #0x30]
Target 0: (Music) stopped.
(lldb) po $x0
<UISwipeActionPullView: 0x11e097190; cellEdge = UIRectEdgeLeft, actions = <NSArray: 0x2820b5230>>

(lldb) po NSStringFromSelector($x1)
_tappedButton:
```

Every objective C method implicitally starts with 2 parameters: `self`, the instance were calling the method on and `command` the selector we are calling. When we ask $x0 is class and $x1 is the function. The [UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller) is responsible for handling the logic when users perform actions in the UI. To find the UIViewController for our UISwipeActionPullView we can start by looking at the nextResponder in lldb

```
(lldb) po [$x0 nextResponder]
<UICollectionView: 0x1048ed600; frame = (0 0; 375 667); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x2820ba6d0>; layer = <CALayer: 0x282e3a920>; contentOffset: {0, 3739.5}; contentSize: {375, 6398}; adjustedContentInset: {64, 0, 113, 0}; layout: <_TtCC16MusicApplication19SongsViewControllerP33_A45DAA2A24828EFCFB77323F0B26236B11TableLayout: 0x11e03dfa0>; dataSource: <MusicApplication.SongsViewController: 0x1048d5c00>>
```
The `recursiveDescription` of keyWindow also showed that this `UICollectionView` was the parent view. Every `UICollectionView` has a delegate which follows the [UICollectionViewDelegate](https://developer.apple.com/documentation/uikit/uicollectionviewdelegate) protocol. We can use lldb to see the delegate of our UICollectionView and to describe what functions the delegate implements.

```
(lldb) po [$x0 nextResponder] delegate]
note: object description requested, but type doesn't implement a custom object description. Consider using "p" instead of "po" (this note will only be shown once per debug session).
<MusicApplication.SongsViewController: 0x10682c800>
(lldb) po [@"" __methodDescriptionForClass:(Class)NSClassFromString(@"MusicApplication.SongsViewController")]

in MusicApplication.SongsViewController:
	Properties:
		@property (nonatomic, readonly) BOOL canBecomeFirstResponder;
	Instance Methods:
		- (id) init; (0x1037070c8)
		- (id) initWithCoder:(id)arg1; (0x1037070e8)
		- (void) viewWillAppear:(BOOL)arg1; (0x1037082bc)
		- (void) viewWillDisappear:(BOOL)arg1; (0x1037082cc)
		- (void) viewDidAppear:(BOOL)arg1; (0x1037084e0)
		- (void) viewDidLoad; (0x10370985c)
		- (void) traitCollectionDidChange:(id)arg1; (0x103709f78)
		- (void) viewWillTransitionToSize:(struct CGSize)arg1 withTransitionCoordinator:(id)arg2; (0x103709fc8)
		- (void) viewDidLayoutSubviews; (0x10370a1e4)
		- (BOOL) canBecomeFirstResponder; (0x10370a210)
		- (long) numberOfSectionsInCollectionView:(id)arg1; (0x10370a218)
		- (long) collectionView:(id)arg1 numberOfItemsInSection:(long)arg2; (0x10370a270)
		- (id) collectionView:(id)arg1 cellForItemAtIndexPath:(id)arg2; (0x10370a99c)
		- (id) collectionView:(id)arg1 viewForSupplementaryElementOfKind:(id)arg2 atIndexPath:(id)arg3; (0x10370b540)
		- (id) _sectionIndexTitlesForCollectionView:(id)arg1; (0x10370b66c)
		- (id) _collectionView:(id)arg1 indexPathForSectionIndexTitle:(id)arg2 atIndex:(long)arg3; (0x10370b878)
		- (void) collectionView:(id)arg1 willPerformPreviewActionForMenuWithConfiguration:(id)arg2 animator:(id)arg3; (0x10370bcf0)
		- (id) collectionView:(id)arg1 contextMenuConfigurationForItemAtIndexPath:(id)arg2 point:(struct CGPoint)arg3; (0x10370bd7c)
		- (id) collectionView:(id)arg1 tableLayout:(id)arg2 leadingSwipeActionsConfigurationForRowAtIndexPath:(id)arg3; (0x10370be64)
		- (id) collectionView:(id)arg1 tableLayout:(id)arg2 trailingSwipeActionsConfigurationForRowAtIndexPath:(id)arg3; (0x10370bf68)
		- (void) collectionView:(id)arg1 willEndContextMenuInteractionWithConfiguration:(id)arg2 animator:(id)arg3; (0x10370c3c0)
		- (void) collectionView:(id)arg1 willDisplayCell:(id)arg2 forItemAtIndexPath:(id)arg3; (0x10370c8c8)
		- (BOOL) collectionView:(id)arg1 shouldSelectItemAtIndexPath:(id)arg2; (0x10370c9cc)
		- (void) collectionView:(id)arg1 didSelectItemAtIndexPath:(id)arg2; (0x10370d234)
		- (double) collectionView:(id)arg1 heightForHeaderViewInTableLayout:(id)arg2; (0x10370d314)
		- (id) initWithCollectionViewLayout:(id)arg1; (0x10370e724)
		- (id) initWithNibName:(id)arg1 bundle:(id)arg2; (0x10370e758)
		- (void) .cxx_destruct; (0x10370e798)

(lldb)
```
The `leadingSwipeActionsConfigurationForRowAtIndexPath` looks like it might be triggered when swiping to queue so I set a breakpoint here to see where it gets triggered. The breakpoint got triggered when I performed the action. This is before the play next and play last buttons are rendered in the view.
```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 11.1
    frame #0: 0x000000010370be64 MusicApplication`___lldb_unnamed_symbol42258
MusicApplication`___lldb_unnamed_symbol42258:
->  0x10370be64 <+0>:  stp    x28, x27, [sp, #-0x60]!
    0x10370be68 <+4>:  stp    x26, x25, [sp, #0x10]
    0x10370be6c <+8>:  stp    x24, x23, [sp, #0x20]
    0x10370be70 <+12>: stp    x22, x21, [sp, #0x30]
Target 0: (Music) stopped.
(lldb) breakpoint list^C
(lldb) po $x0
<MusicApplication.SongsViewController: 0x1048d5c00>

(lldb) po po NSStringFromSelector($x1)^C
(lldb) po NSStringFromSelector($x1)
collectionView:tableLayout:leadingSwipeActionsConfigurationForRowAtIndexPath:

(lldb) po $x2
<UICollectionView: 0x1048ed600; frame = (0 0; 375 667); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x2820ba6d0>; layer = <CALayer: 0x282e3a920>; contentOffset: {0, 5223.5}; contentSize: {375, 6398}; adjustedContentInset: {64, 0, 113, 0}; layout: <_TtCC16MusicApplication19SongsViewControllerP33_A45DAA2A24828EFCFB77323F0B26236B11TableLayout: 0x11e03dfa0>; dataSource: <MusicApplication.SongsViewController: 0x1048d5c00>>

(lldb) po $x3
<_TtCC16MusicApplication19SongsViewControllerP33_A45DAA2A24828EFCFB77323F0B26236B11TableLayout: 0x11e03dfa0>

(lldb) po $x4
<NSIndexPath: 0x94e2caae6068a274> {length = 2, path = 6 - 25}

```
This function is also documented on Apple's website at[tableView:leadingSwipeActionsConfigurationForRowAtIndexPath:](https://developer.apple.com/documentation/uikit/uitableviewdelegate/2902366-tableview?language=objc). [NSIndexPath](https://developer.apple.com/documentation/foundation/nsindexpath?language=objc) is list of indexes that together represent the path to a specific location in a tree of nested arrays. From the documentation we see the return value is a `UISwipeActionsConfiguration` We can also use lldb to see the return value of the function. I used `di` to disassemble the current function and set a breakpoint on the last line of the function. When the breakpoint got triggered this is just before function
```
(lldb) po $x0
<UISwipeActionsConfiguration: 0x281149140: actions=(
    "<UISwipeAction: 0x283bcaf40: style=0, title=Playing Next, backgroundColor=<UIDynamicSystemColor: 0x28053ed80; name = systemIndigoColor>, image=<UIImage:0x282d48ea0 symbol(com.apple.MusicApplication: play.next) {21.5, 16.5} baseline=2,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-1.5, 0, -2, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>",
    "<UISwipeAction: 0x283bca760: style=0, title=Playing Last, backgroundColor=<UIDynamicSystemColor: 0x28053f280; name = systemOrangeColor>, image=<UIImage:0x282d48120 symbol(com.apple.MusicApplication: play.last) {21.5, 16.5} baseline=2.5,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-2, 0, -1.5, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>"
), performsFirstActionWithFullSwipe=1>
```

I still need to understand the process of creating a `UISwipeActionsConfiguration` and figure out where the actual queing happens so I open this function in a disassember. The address for the app in Memory is always randomized because of [ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization). With lldb we can see exactly what memory address of the music binary this function is at
```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000105857e64 MusicApplication`___lldb_unnamed_symbol42258
MusicApplication`___lldb_unnamed_symbol42258:
->  0x105857e64 <+0>:  stp    x28, x27, [sp, #-0x60]!
    0x105857e68 <+4>:  stp    x26, x25, [sp, #0x10]
    0x105857e6c <+8>:  stp    x24, x23, [sp, #0x20]
    0x105857e70 <+12>: stp    x22, x21, [sp, #0x30]
Target 0: (Music) stopped.
(lldb) image lookup -a 0x0000000105857e64
      Address: MusicApplication[0x0000000000367e64] (MusicApplication.__TEXT.__text + 3551108)
      Summary: MusicApplication`___lldb_unnamed_symbol42258
```
This means that this function exists at `0x0000000000367e64` in the Music binary. We can also disassemble the current function with `di` in lldb. 

```
(lldb) di
MusicApplication`___lldb_unnamed_symbol42258:
->  0x105857e64 <+0>:   stp    x28, x27, [sp, #-0x60]!
    0x105857e68 <+4>:   stp    x26, x25, [sp, #0x10]
    0x105857e6c <+8>:   stp    x24, x23, [sp, #0x20]
    0x105857e70 <+12>:  stp    x22, x21, [sp, #0x30]
    0x105857e74 <+16>:  stp    x20, x19, [sp, #0x40]
    0x105857e78 <+20>:  stp    x29, x30, [sp, #0x50]
    0x105857e7c <+24>:  add    x29, sp, #0x50
    0x105857e80 <+28>:  mov    x19, x4
    0x105857e84 <+32>:  mov    x20, x3
    0x105857e88 <+36>:  mov    x21, x2
    0x105857e8c <+40>:  mov    x22, x0
    0x105857e90 <+44>:  mov    x0, #0x0
    0x105857e94 <+48>:  bl     0x105eb049c
    0x105857e98 <+52>:  mov    x23, x0
    0x105857e9c <+56>:  ldur   x27, [x0, #-0x8]
    0x105857ea0 <+60>:  ldr    x8, [x27, #0x40]
    0x105857ea4 <+64>:  mov    x9, x8
    0x105857ea8 <+68>:  adrp   x16, 2162
    0x105857eac <+72>:  ldr    x16, [x16, #0xff8]
    0x105857eb0 <+76>:  blr    x16
    0x105857eb4 <+80>:  mov    x9, sp
    0x105857eb8 <+84>:  add    x8, x8, #0xf
    0x105857ebc <+88>:  and    x8, x8, #0xfffffffffffffff0
    0x105857ec0 <+92>:  sub    x24, x9, x8
    0x105857ec4 <+96>:  mov    sp, x24
    0x105857ec8 <+100>: mov    x8, x24
    0x105857ecc <+104>: mov    x0, x19
    0x105857ed0 <+108>: bl     0x105eb043c
    0x105857ed4 <+112>: mov    x0, x22
    0x105857ed8 <+116>: bl     0x105eb38bc               ; symbol stub for: objc_retain
    0x105857edc <+120>: mov    x25, x0
    0x105857ee0 <+124>: mov    x0, x21
    0x105857ee4 <+128>: bl     0x105eb38bc               ; symbol stub for: objc_retain
    0x105857ee8 <+132>: mov    x21, x0
    0x105857eec <+136>: mov    x0, x20
    0x105857ef0 <+140>: bl     0x105eb38bc               ; symbol stub for: objc_retain
    0x105857ef4 <+144>: mov    x26, x0
    0x105857ef8 <+148>: mov    x0, x19
    0x105857efc <+152>: bl     0x105eb38bc               ; symbol stub for: objc_retain
    0x105857f00 <+156>: mov    x19, x0
    0x105857f04 <+160>: mov    x0, x24
    0x105857f08 <+164>: mov    x20, x22
    0x105857f0c <+168>: bl     0x10585cd34               ; ___lldb_unnamed_symbol42327
    0x105857f10 <+172>: mov    x20, x0
    0x105857f14 <+176>: ldr    x8, [x27, #0x8]
    0x105857f18 <+180>: mov    x0, x24
    0x105857f1c <+184>: mov    x1, x23
    0x105857f20 <+188>: blr    x8
    0x105857f24 <+192>: mov    x0, x21
    0x105857f28 <+196>: bl     0x105eb38b0               ; symbol stub for: objc_release
    0x105857f2c <+200>: mov    x0, x26
    0x105857f30 <+204>: bl     0x105eb38b0               ; symbol stub for: objc_release
    0x105857f34 <+208>: mov    x0, x25
    0x105857f38 <+212>: bl     0x105eb38b0               ; symbol stub for: objc_release
    0x105857f3c <+216>: mov    x0, x19
    0x105857f40 <+220>: bl     0x105eb38b0               ; symbol stub for: objc_release
    0x105857f44 <+224>: mov    x0, x20
    0x105857f48 <+228>: sub    sp, x29, #0x50
    0x105857f4c <+232>: ldp    x29, x30, [sp, #0x50]
    0x105857f50 <+236>: ldp    x20, x19, [sp, #0x40]
    0x105857f54 <+240>: ldp    x22, x21, [sp, #0x30]
    0x105857f58 <+244>: ldp    x24, x23, [sp, #0x20]
    0x105857f5c <+248>: ldp    x26, x25, [sp, #0x10]
    0x105857f60 <+252>: ldp    x28, x27, [sp], #0x60
    0x105857f64 <+256>: b      0x105eb37f0               ; symbol stub for: objc_autoreleaseReturnValue
```

I am curious about the function call to `___lldb_unnamed_symbol42327` so I set a breakpoint after it returns to check what is returned.
```
(lldb) b -a 0x105857f10
Breakpoint 16: where = MusicApplication`___lldb_unnamed_symbol42258 + 172, address = 0x0000000105857f10
(lldb) c
Process 38799 resuming
Process 38799 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 16.1
    frame #0: 0x0000000105857f10 MusicApplication`___lldb_unnamed_symbol42258 + 172
MusicApplication`___lldb_unnamed_symbol42258:
->  0x105857f10 <+172>: mov    x20, x0
    0x105857f14 <+176>: ldr    x8, [x27, #0x8]
    0x105857f18 <+180>: mov    x0, x24
    0x105857f1c <+184>: mov    x1, x23
Target 0: (Music) stopped.
(lldb) po $x0
<UISwipeActionsConfiguration: 0x28261d6e0: actions=(
    "<UISwipeAction: 0x280d37c00: style=0, title=Playing Next, backgroundColor=<UIDynamicSystemColor: 0x2833c6900; name = systemIndigoColor>, image=<UIImage:0x281baeeb0 symbol(com.apple.MusicApplication: play.next) {21.5, 16.5} baseline=2,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-1.5, 0, -2, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>",
    "<UISwipeAction: 0x280d34180: style=0, title=Playing Last, backgroundColor=<UIDynamicSystemColor: 0x2833c6e00; name = systemOrangeColor>, image=<UIImage:0x281bad3b0 symbol(com.apple.MusicApplication: play.last) {21.5, 16.5} baseline=2.5,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-2, 0, -1.5, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>"
), performsFirstActionWithFullSwipe=1>
```
Notice how this is the same thing that the original function is returning. I am guessing the original function is just a wrapper for `___lldb_unnamed_symbol42327` which does memory management. The pseudocode of setup looks like this


```python
def leadingswipedaction():  # ___lldb_unnamed_symbol42258
    # malloc
    res = create_UISwipeActionsConfiguration() # ___lldb_unnamed_symbol42327
    # free memory
    return res

```
We still need to understand how the inner `create_UISwipeActionsConfiguration` call is being made. I was also able to find documentation for [UISwipeActionsConfiguration](https://developer.apple.com/documentation/uikit/uiswipeactionsconfiguration). The documentation shows there is an init function which takes a list of actions. These actions look similar to the `UISwipeAction` however the documentation shows the type is of `UIContextualAction`. We can use lldb to check if our `UISwipeAction` has a super class

```
(lldb) po $x0
<UISwipeActionsConfiguration: 0x283b4ccf0: actions=(
    "<UISwipeAction: 0x281244180: style=0, title=Playing Next, backgroundColor=<UIDynamicSystemColor: 0x282d031c0; name = systemIndigoColor>, image=<UIImage:0x28056e910 symbol(com.apple.MusicApplication: play.next) {21.5, 16.5} baseline=2,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-1.5, 0, -2, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>",
    "<UISwipeAction: 0x281244d20: style=0, title=Playing Last, backgroundColor=<UIDynamicSystemColor: 0x282d036c0; name = systemOrangeColor>, image=<UIImage:0x28056f600 symbol(com.apple.MusicApplication: play.last) {21.5, 16.5} baseline=2.5,contentInsets={1, 0.5, 1, 1},alignmentRectInsets={-2, 0, -1.5, 0} config=<traits=(UserInterfaceIdiom = Phone, DisplayScale = 2, DisplayGamut = P3, HorizontalSizeClass = Compact, VerticalSizeClass = Regular, UserInterfaceStyle = Dark, UserInterfaceLayoutDirection = LTR, PreferredContentSizeCategory = L, AccessibilityContrast = Normal)> renderingMode=automatic>>"
), performsFirstActionWithFullSwipe=1>

(lldb) po [0x281244180 superclass]
UIContextualAction
```

The `create_UISwipeActionsConfiguration` is probably creating two UIContextualAction for the actions array. The documentation for [UIContextualAction](https://developer.apple.com/documentation/uikit/uicontextualaction) shows there is an init function which takes a handler as a parameter. the handler is the third argument.

ss-----------------------------------------
```
MediaPlaybackCore`+[MPCMediaRemoteController _sendLocalCommand:playbackIntent:options:toPlayerPath:completion:]:
->  0x1b027dea4 <+0>:  sub    sp, sp, #0xb0
    0x1b027dea8 <+4>:  stp    x26, x25, [sp, #0x60]
    0x1b027deac <+8>:  stp    x24, x23, [sp, #0x70]
    0x1b027deb0 <+12>: stp    x22, x21, [sp, #0x80]
Target 0: (Music) stopped.
(lldb) po $x0
MPCMediaRemoteController

(lldb) po NSStringFromSelector($x1)
_sendLocalCommand:playbackIntent:options:toPlayerPath:completion:

(lldb) po $x2
125

(lldb) po $x3
<MPCPlaybackIntent: 0x281970280 source=3>

(lldb) po $x4
{
    MPCPlayerCommandRequestMediaRemoteOptionPlaybackIntent = "<MPCPlaybackIntent: 0x281970280 source=3>";
    kMRMediaRemoteOptionAssistantCommandSendTimestamp = "739667642.7544";
    kMRMediaRemoteOptionNowPlayingContentItemID = "XgeEllFNg\U2206wXe84XFkw";
    kMRMediaRemoteOptionPlaybackQueueInsertionPosition = 0;
    kMRMediaRemoteOptionRemoteControlInterfaceIdentifier = "Player.DataSource.PlaybackCommand";
}

<MPInsertIntoPlaybackQueueCommand: 0x281c59810 type=InsertIntoPlaybackQueue (125) enabled=YES handlers=[0x281c36760:_performCommandEvent:completion:]>

+[MPRemoteCommandEvent eventWithCommand:mediaRemoteType:options:]
-[MPRemoteCommandEvent initWithCommand:mediaRemoteType:options:]
```
```
Process 67952 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 15.1
    frame #0: 0x00000001b0138c40 MediaPlaybackCore`-[MPCPlayerRequest playingItemProperties]
MediaPlaybackCore`-[MPCPlayerRequest playingItemProperties]:
->  0x1b0138c40 <+0>:  adrp   x8, 295481
    0x1b0138c44 <+4>:  ldrsw  x8, [x8, #0xdd0]
    0x1b0138c48 <+8>:  ldr    x0, [x0, x8]
    0x1b0138c4c <+12>: ret
Target 0: (Music) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 15.1
  * frame #0: 0x00000001b0138c40 MediaPlaybackCore`-[MPCPlayerRequest playingItemProperties]
    frame #1: 0x00000001b0134588 MediaPlaybackCore`-[_MPCPlayerResponseTracklistDataSource itemAtIndexPath:] + 208
    frame #2: 0x00000001897f3530 MediaPlayer`-[MPLazySectionedCollection itemAtIndexPath:] + 236
    frame #3: 0x00000001b01385e0 MediaPlaybackCore`-[MPCPlayerResponseTracklist playingItem] + 32
    frame #4: 0x00000001b0235c10 MediaPlaybackCore`-[_MPCPlayerInsertItemsCommand insertAfterPlayingItemWithPlaybackIntent:] + 808
    frame #5: 0x000000010259ae00 MusicApplication`___lldb_unnamed_symbol66873 + 132
    frame #6: 0x0000000102599d18 MusicApplication`___lldb_unnamed_symbol66821 + 304
    frame #7: 0x000000010258ab00 MusicApplication`___lldb_unnamed_symbol66530
    frame #8: 0x0000000102588d18 MusicApplication`___lldb_unnamed_symbol66519
    frame #9: 0x0000000101e141dc MusicApplication`___lldb_unnamed_symbol25420
    frame #10: 0x0000000102430b28 MusicApplication`___lldb_unnamed_symbol57987
    frame #11: 0x0000000102438684 MusicApplication`___lldb_unnamed_symbol58139
    frame #12: 0x0000000101e4dbf4 MusicApplication`___lldb_unnamed_symbol26505
    frame #13: 0x0000000102438660 MusicApplication`___lldb_unnamed_symbol58130
```


```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 9.1
    frame #0: 0x00000001b012c5b4 MediaPlaybackCore`-[MPCPlayerCommandRequest initWithMediaRemoteCommand:options:controller:label:]
MediaPlaybackCore`-[MPCPlayerCommandRequest initWithMediaRemoteCommand:options:controller:label:]:
->  0x1b012c5b4 <+0>:  stp    x26, x25, [sp, #-0x50]!
    0x1b012c5b8 <+4>:  stp    x24, x23, [sp, #0x10]
    0x1b012c5bc <+8>:  stp    x22, x21, [sp, #0x20]
    0x1b012c5c0 <+12>: stp    x20, x19, [sp, #0x30]
Target 0: (Music) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 9.1
  * frame #0: 0x00000001b012c5b4 MediaPlaybackCore`-[MPCPlayerCommandRequest initWithMediaRemoteCommand:options:controller:label:]
    frame #1: 0x00000001b02361e0 MediaPlaybackCore`-[_MPCPlayerInsertItemsCommand _insertWithOptions:] + 392
    frame #2: 0x00000001b0235c84 MediaPlaybackCore`-[_MPCPlayerInsertItemsCommand insertAfterPlayingItemWithPlaybackIntent:] + 924
    frame #3: 0x00000001061dee00 MusicApplication`___lldb_unnamed_symbol66873 + 132
    frame #4: 0x00000001061ddd18 MusicApplication`___lldb_unnamed_symbol66821 + 304
    frame #5: 0x00000001061ceb00 MusicApplication`___lldb_unnamed_symbol66530
    frame #6: 0x00000001061ccd18 MusicApplication`___lldb_unnamed_symbol66519
    frame #7: 0x0000000105a581dc MusicApplication`___lldb_unnamed_symbol25420
    frame #8: 0x0000000106074b28 MusicApplication`___lldb_unnamed_symbol57987
    frame #9: 0x000000010607c684 MusicApplication`___lldb_unnamed_symbol58139
    frame #10: 0x0000000105a91bf4 MusicApplication`___lldb_unnamed_symbol26505
    frame #11: 0x000000010607c660 MusicApplication`___lldb_unnamed_symbol58130
(lldb) po $x0
```
```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 12.1
    frame #0: 0x00000001b01e0e20 MediaPlaybackCore`-[MPCPlayerChangeRequest initWithCommandRequests:]
MediaPlaybackCore`-[MPCPlayerChangeRequest initWithCommandRequests:]:
->  0x1b01e0e20 <+0>:  sub    sp, sp, #0x30
    0x1b01e0e24 <+4>:  stp    x20, x19, [sp, #0x10]
    0x1b01e0e28 <+8>:  stp    x29, x30, [sp, #0x20]
    0x1b01e0e2c <+12>: add    x29, sp, #0x20
Target 0: (Music) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 12.1
  * frame #0: 0x00000001b01e0e20 MediaPlaybackCore`-[MPCPlayerChangeRequest initWithCommandRequests:]
    frame #1: 0x00000001025a1e98 MusicApplication`___lldb_unnamed_symbol66821 + 688
    frame #2: 0x0000000102592b00 MusicApplication`___lldb_unnamed_symbol66530
    frame #3: 0x0000000102590d18 MusicApplication`___lldb_unnamed_symbol66519
    frame #4: 0x0000000101e1c1dc MusicApplication`___lldb_unnamed_symbol25420
    frame #5: 0x0000000102438b28 MusicApplication`___lldb_unnamed_symbol57987
    frame #6: 0x0000000102440684 MusicApplication`___lldb_unnamed_symbol58139
    frame #7: 0x0000000101e55bf4 MusicApplication`___lldb_unnamed_symbol26505
    frame #8: 0x0000000102440660 MusicApplication`___lldb_unnamed_symbol58130
```

```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 5.1
    frame #0: 0x000000018262eb04 Foundation`-[NSOperationQueue addOperation:]
Foundation`-[NSOperationQueue addOperation:]:
->  0x18262eb04 <+0>: mov    x3, #0x0
    0x18262eb08 <+4>: mov    w4, #0x0
    0x18262eb0c <+8>: b      0x182591a14               ; __addOperations

Foundation`:
    0x18262eb10 <+0>: stp    x20, x19, [sp, #-0x20]!
Target 0: (Music) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 5.1
  * frame #0: 0x000000018262eb04 Foundation`-[NSOperationQueue addOperation:]
    frame #1: 0x00000001b01e1740 MediaPlaybackCore`-[MPCPlayerChangeRequest performWithExtendedStatusCompletion:] + 1592
    frame #2: 0x00000001061ddf68 MusicApplication`___lldb_unnamed_symbol66821 + 896
    frame #3: 0x00000001061ceb00 MusicApplication`___lldb_unnamed_symbol66530
    frame #4: 0x00000001061ccd18 MusicApplication`___lldb_unnamed_symbol66519
    frame #5: 0x0000000105a581dc MusicApplication`___lldb_unnamed_symbol25420
    frame #6: 0x0000000106074b28 MusicApplication`___lldb_unnamed_symbol57987
    frame #7: 0x000000010607c684 MusicApplication`___lldb_unnamed_symbol58139
    frame #8: 0x0000000105a91bf4 MusicApplication`___lldb_unnamed_symbol26505
    frame #9: 0x000000010607c660 MusicApplication`___lldb_unnamed_symbol58130
(lldb)
```

```
(lldb) bt
* thread #36, queue = 'NSOperationQueue 0x113fb2cb0 (QOS: USER_INITIATED)', stop reason = breakpoint 4.1
  * frame #0: 0x00000001b027dea4 MediaPlaybackCore`+[MPCMediaRemoteController _sendLocalCommand:playbackIntent:options:toPlayerPath:completion:]
    frame #1: 0x00000001b027db0c MediaPlaybackCore`+[MPCMediaRemoteController sendCommand:options:toPlayerPath:completion:] + 224
    frame #2: 0x00000001b027a264 MediaPlaybackCore`-[MPCMediaRemoteController sendCommand:options:completion:] + 448
    frame #3: 0x00000001b01e21a8 MediaPlaybackCore`-[MPCMediaRemoteCommandOperation execute] + 1860
    frame #4: 0x000000018982c730 MediaPlayer`-[MPAsyncOperation start] + 148
    frame #5: 0x00000001825ba498 Foundation`__NSOPERATIONQUEUE_IS_STARTING_AN_OPERATION__ + 20
    frame #6: 0x00000001825c7ccc Foundation`__NSOQSchedule_f + 180
    frame #7: 0x0000000180bab094 libdispatch.dylib`_dispatch_call_block_and_release + 24
    frame #8: 0x0000000180bac094 libdispatch.dylib`_dispatch_client_callout + 16
    frame #9: 0x0000000180b4ebb8 libdispatch.dylib`_dispatch_continuation_pop$VARIANT$mp + 440
    frame #10: 0x0000000180b4e2e0 libdispatch.dylib`_dispatch_async_redirect_invoke + 588
    frame #11: 0x0000000180b5bb94 libdispatch.dylib`_dispatch_root_queue_drain + 340
    frame #12: 0x0000000180b5c39c libdispatch.dylib`_dispatch_worker_thread2 + 172
    frame #13: 0x00000001dc250dc4 libsystem_pthread.dylib`_pthread_wqthread + 224
```

```
Process 56769 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 12.1
    frame #0: 0x0000000189905094 MediaPlayer`+[MPRemoteCommandEvent eventWithCommand:mediaRemoteType:options:]
MediaPlayer`+[MPRemoteCommandEvent eventWithCommand:mediaRemoteType:options:]:
->  0x189905094 <+0>:  stp    x22, x21, [sp, #-0x30]!
    0x189905098 <+4>:  stp    x20, x19, [sp, #0x10]
    0x18990509c <+8>:  stp    x29, x30, [sp, #0x20]
    0x1899050a0 <+12>: add    x29, sp, #0x20
Target 0: (Music) stopped.
(lldb) po $x0
MPRemoteCommandEvent

(lldb) po NSStringFromSelector($x1)
eventWithCommand:mediaRemoteType:options:

(lldb) po $x3
125

(lldb) po $x4
{
    MPCPlayerCommandRequestMediaRemoteOptionPlaybackContext = "<MPCModelPlaybackContext:0x283dd0e40 actionAfterQueueLoad=Play playbackRequestEnvironment=<MPCPlaybackRequestEnvironment: 0x28127d440 identity=<ICUserIdentity 0x280485ce0: [Active Account: <ef30d873b760aab549b318eb2bd03209120026a5>]>> queueEndAction=UserDefault[Reset] repeat/shuffle=UserDefault[Off]/UserDefault[Off] request=<MPModelLibraryRequest: 0x2839c9c20 label=\U201cSong: Optional(\"CHIHIRO\")\U201c itemKind=\U201csongs\U201d allowedItemIdentifiers=[(databaseID=\"8ABAD0A8-598A-49E1-AEB9-3C90AFF32057\" personID=\"25116999996\" persistentID=3325340846184242708 storeCloudAlbumID=\"l.SKGOG9t\" storeCloudID=182889506 cloudUniversalLibraryID=\"i.mmpe6Q9iLDeXolW\" storeSubscriptionAdamID=1739659141 reportingAdamID=1739659141 assetAdamID=1739659141 kind=songs)]> startItemIdentifiers=(databaseID=\"8ABAD0A8-598A-49E1-AEB9-3C90AFF32057\" personID=\"25116999996\" persistentID=3325340846184242708 storeCloudAlbumID=\"l.SKGOG9t\" storeCloudID=182889506 cloudUniversalLibraryID=\"i.mmpe6Q9iLDeXolW\" storeSubscriptionAdamID=1739659141 reportingAdamID=1739659141 assetAdamID=1739659141 kind=songs)>";
    kMRMediaRemoteOptionAssistantCommandSendTimestamp = "739698003.02045";
    kMRMediaRemoteOptionCommandID = "63A1B5E2-0D0E-497E-BCC9-89247B1B20CA";
    kMRMediaRemoteOptionNowPlayingContentItemID = "g8SkwnkEk\U2206w8XHXXS6n";
    kMRMediaRemoteOptionPlaybackQueueInsertionPosition = 0;
    kMRMediaRemoteOptionRemoteControlInterfaceIdentifier = "Player.DataSource.PlaybackCommand";
    kMRMediaRemoteOptionSendOptionsNumber = 0;
    kMRMediaRemoteOptionSenderID = "SenderDevice = <Rahul\U2019s Developer iPhone>, SenderBundleIdentifier = <com.apple.Music>, SenderPID = <56769>";
    kMRMediaRemoteOptionSystemAppPlaybackQueueData = {length = 143, bytes = 0x62706c69 73743030 d5010203 04050607 ... 00000000 00000064 };
}

(lldb) expr -l objc++ -O -- [$x4 setValue:@(2) forKey:@"kMRMediaRemoteOptionPlaybackQueueInsertionPosition"]
0

(lldb) c
Process 56769 resuming
```

[MediaPlaybackCore](https://developer.limneos.net/?ios=14.4&framework=MediaPlaybackCore.framework&header=MediaPlaybackCore.h)

-[MPCQueueController moveContentItemID:afterContentItemID:completion:]


```
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 13.4
  * frame #0: 0x00000001b0239260 MediaPlaybackCore`-[_MPCPlayerReorderItemsCommand moveItem:afterItem:]
    frame #1: 0x0000000101ecccf4 MusicApplication`___lldb_unnamed_symbol29077 + 1236
    frame #2: 0x0000000101ebfff0 MusicApplication`___lldb_unnamed_symbol28973 + 296
    frame #3: 0x00000001839d02b8 UIKitCore`-[UICollectionView _notifyDataSourceForMoveOfItemFromIndexPath:toIndexPath:] + 276
    frame #4: 0x00000001839cf3bc UIKitCore`-[UICollectionView _completeInteractiveMovementWithDisposition:completion:] + 1332
    frame #5: 0x0000000101ebee80 MusicApplication`___lldb_unnamed_symbol28954 + 184
    frame #6: 0x0000000101ebf020 MusicApplication`___lldb_unnamed_symbol28955 + 28
    frame #7: 0x00000001839a5604 UIKitCore`-[UICollectionViewTableCell _endReorderingForCell:wasCancelled:animated:] + 100
    frame #8: 0x000000018414113c UIKitCore`-[UITableViewCell _grabberReleased:] + 64
    frame #9: 0x0000000184147558 UIKitCore`-[UITableViewCellReorderControl endTrackingWithTouch:withEvent:] + 68
    frame #10: 0x00000001837f147c UIKitCore`-[UIControl touchesEnded:withEvent:] + 448
    frame #11: 0x00000001832f717c UIKitCore`-[UIWindow _sendTouchesForEvent:] + 1228
    frame #12: 0x0000000183326c9c UIKitCore`-[UIWindow sendEvent:] + 4372
    frame #13: 0x000000010228bd08 MusicApplication`___lldb_unnamed_symbol49000 + 84
    frame #14: 0x00000001834c7ab4 UIKitCore`-[UIApplication sendEvent:] + 892
    frame #15: 0x00000001832fbc7c UIKitCore`__dispatchPreprocessedEventFromEventQueue + 8148
    frame #16: 0x00000001832f0b40 UIKitCore`__processEventQueue + 6544
    frame #17: 0x00000001832f5f5c UIKitCore`__eventFetcherSourceCallback + 168
    frame #18: 0x0000000180f0c468 CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 24
    frame #19: 0x0000000180f1c598 CoreFoundation`__CFRunLoopDoSource0 + 204
    frame #20: 0x0000000180e5e774 CoreFoundation`__CFRunLoopDoSources0 + 256
    frame #21: 0x0000000180e63e48 CoreFoundation`__CFRunLoopRun + 768
    frame #22: 0x0000000180e77194 CoreFoundation`CFRunLoopRunSpecific + 572
    frame #23: 0x00000001a19a2988 GraphicsServices`GSEventRunModal + 160
    frame #24: 0x000000018367aa88 UIKitCore`-[UIApplication _run] + 1080
    frame #25: 0x0000000183413fc8 UIKitCore`UIApplicationMain + 336
    frame #26: 0x0000000101e144b0 MusicApplication`main + 68
    frame #27: 0x0000000100dac4d0 dyld`start + 444
(lldb)
```
ss-----------------------------------------

This shows where in memory these binary functions are loaded. These functions most likely don't get loaded until they are opened which is why we see errors if we try running this without loading

```
(lldb) b -a 0x101ca01a8
Breakpoint 1: where = MusicApplication`___lldb_unnamed_symbol29133, address = 0x0000000101ca01a8
(lldb) con
Process 20285 resuming
Process 20285 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000101ca01a8 MusicApplication`___lldb_unnamed_symbol29133
MusicApplication`___lldb_unnamed_symbol29133:
->  0x101ca01a8 <+0>:  stp    x28, x27, [sp, #-0x60]!
    0x101ca01ac <+4>:  stp    x26, x25, [sp, #0x10]
    0x101ca01b0 <+8>:  stp    x24, x23, [sp, #0x20]
    0x101ca01b4 <+12>: stp    x22, x21, [sp, #0x30]
Target 0: (Music) stopped.
(lldb) po $x0
<MusicApplication.NowPlayingQueueViewController: 0x101165400>
(lldb) po $x1
6894563425
(lldb) po NSStringFromSelector($x1)
collectionView:targetIndexPathForMoveFromItemAtIndexPath:toProposedIndexPath:
```



To get stopped in this debugger I was moving the song in the second position to first position. Let's take a look at the variables that were passed. There are 3 variables here`arg1 targetIndexPathForMoveFromItemAtIndexPath:(id)arg2 toProposedIndexPath:(id)arg3` Since `$arg1` and `$arg2` are the implicit arguments lets look at `$arg3`

```
(lldb) po $arg3
<_TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView: 0x101404000; baseClass = UICollectionView; frame = (0 0; 375 539); autoresize = W+H; gestureRecognizers = <NSArray: 0x28023ee80>; layer = <CALayer: 0x280faa480>; contentOffset: {0, 1414}; contentSize: {375, 1744}; adjustedContentInset: {0, 0, 209, 0}; layout: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE619CompositionalLayout: 0x122dadb20>; dataSource: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource: 0x28023fae0>>

(lldb) po [@"" __methodDescriptionForClass:[_TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView class]]

in _TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView:
	Instance Methods:
		- (BOOL) beginInteractiveMovementForItemAtIndexPath:(id)arg1; (0x101c8ad08)
		- (void) endInteractiveMovement; (0x101c8b004)
		- (void) cancelInteractiveMovement; (0x101c8b0ac)
		- (id) initWithFrame:(struct CGRect)arg1 collectionViewLayout:(id)arg2; (0x101c8b0d8)
		- (id) initWithCoder:(id)arg1; (0x101c8b1ac)
		- (void) .cxx_destruct; (0x101c8b204)

(lldb)
```

I set a breakpoint at `beginInteractiveMovementForItemAtIndexPath` and continued execution. This breakpoint did not get called. However after reordering the queue again I saw breakpoint `beginInteractiveMovementForItemAtIndexPath` is called before any calls to `targetIndexPathForMoveFromItemAtIndexPath` are made. We know the structure looks something like this

```
beginInteractiveMovementForItemAtIndexPath(arg1)
targetIndexPathForMoveFromItemAtIndexPath(arg1, arg2, arg3)
targetIndexPathForMoveFromItemAtIndexPath(arg1, arg2, arg3)
targetIndexPathForMoveFromItemAtIndexPath(arg1, arg2, arg3)
```

I've added targetIndexPathForMoveFromItemAtIndexPath 3 times since it gets called multiple times everytime I reorder. I stopped at the `beginInteractiveMovementForItemAtIndexPath` Let's see what the arguments to this function are

```
(lldb) po $x0
<_TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView: 0x1033d6600; baseClass = UICollectionView; frame = (0 0; 375 539); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x28022ad90>; layer = <CALayer: 0x280fd0c20>; contentOffset: {0, 1414}; contentSize: {375, 1912}; adjustedContentInset: {0, 0, 196, 0}; layout: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE619CompositionalLayout: 0x123165f60>; dataSource: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource: 0x280228780>>

(lldb) po NSStringFromSelector($x1)
beginInteractiveMovementForItemAtIndexPath:

(lldb) po $x2
<NSIndexPath: 0x8108f24b6bc48251> {length = 2, path = 1 - 1}
```

I wanted to see how the index paths changed as I changed the queue. The `$x2` above is when 2nd song in queue moves to 1st.. This time I moved 3rd -> 1st and then 1st -> 3rd. This was the result
```
// 3rd -> 1st
<NSIndexPath: 0x8108f24b68c48251> {length = 2, path = 1 - 2}
// 4th -> 1st
<NSIndexPath: 0x8108f24b69c48251> {length = 2, path = 1 - 3}
// 5th -> 1st
<NSIndexPath: 0x8108f24b6ec48251> {length = 2, path = 1 - 4}

// 5th -> 2nd
<NSIndexPath: 0x8108f24b6ec48251> {length = 2, path = 1 - 4}
// 5th -> 3rd
<NSIndexPath: 0x8108f24b6ec48251> {length = 2, path = 1 - 4}

// 1st -> 4th
<NSIndexPath: 0x8108f24b6ac48251> {length = 2, path = 1 - 0}
```
First number is always 1 second number is the index of the song we moved. Next I put a breakpoint on the datasource itself

The ViewController is of type [UICollectionView](https://developer.apple.com/documentation/uikit/uicollectionview). Asking for the `delegate` of the `_TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView` shows the following
```
(lldb) po $arg3
<_TtCC16MusicApplication29NowPlayingQueueViewController14CollectionView: 0x1034a6a00; baseClass = UICollectionView; frame = (0 0; 375 539); autoresize = W+H; gestureRecognizers = <NSArray: 0x280318180>; layer = <CALayer: 0x2808aa1a0>; contentOffset: {0, 1750}; contentSize: {375, 2360}; adjustedContentInset: {0, 0, 0, 0}; layout: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE619CompositionalLayout: 0x124e72d40>; dataSource: <_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource: 0x2803539c0>>

(lldb) po [$arg3 delegate]
<MusicApplication.NowPlayingQueueViewController: 0x103529c00>
(lldb) po [@"" __methodDescriptionForClass:(Class)NSClassFromString(@"MusicApplication.NowPlayingQueueViewController")]

in MusicApplication.NowPlayingQueueViewController:
	Instance Methods:
		- (void) collectionView:(id)arg1 willDisplayContextMenuWithConfiguration:(id)arg2 animator:(id)arg3; (0x105641620)
		- (void) collectionView:(id)arg1 willEndContextMenuInteractionWithConfiguration:(id)arg2 animator:(id)arg3; (0x105641638)
		- (void) collectionView:(id)arg1 willDisplayCell:(id)arg2 forItemAtIndexPath:(id)arg3; (0x10563f690)
		- (void) collectionView:(id)arg1 willDisplaySupplementaryView:(id)arg2 forElementKind:(id)arg3 atIndexPath:(id)arg4; (0x10563f794)
		- (struct CGPoint) collectionView:(id)arg1 targetContentOffsetForProposedContentOffset:(struct CGPoint)arg2; (0x10563fa28)
		- (id) collectionView:(id)arg1 targetIndexPathForMoveFromItemAtIndexPath:(id)arg2 toProposedIndexPath:(id)arg3; (0x1056401a8)
		- (BOOL) collectionView:(id)arg1 shouldSelectItemAtIndexPath:(id)arg2; (0x1056404e8)
		- (void) collectionView:(id)arg1 didSelectItemAtIndexPath:(id)arg2; (0x105640fbc)
		- (void) collectionView:(id)arg1 willPerformPreviewActionForMenuWithConfiguration:(id)arg2 animator:(id)arg3; (0x10564109c)
		- (id) collectionView:(id)arg1 contextMenuConfigurationForItemAtIndexPath:(id)arg2 point:(struct CGPoint)arg3; (0x105641314)
		- (id) collectionView:(id)arg1 previewForHighlightingContextMenuWithConfiguration:(id)arg2; (0x105641414)
		- (id) collectionView:(id)arg1 previewForDismissingContextMenuWithConfiguration:(id)arg2; (0x105641490)
		- (id) _collectionView:(id)arg1 indexPathOfReferenceItemToPreserveContentOffsetWithProposedReference:(id)arg2; (0x1056414b4)
		- (void) configureCell:(id)arg1 forSong:(id)arg2; (0x10563eef0)
		- (void) configureCell:(id)arg1 forTVEpisode:(id)arg2; (0x10563eefc)
		- (void) configureCell:(id)arg1 forMovie:(id)arg2; (0x10563ef08)
		- (void) dealloc; (0x10562dd38)
		- (id) initWithCoder:(id)arg1; (0x10562decc)
		- (void) viewDidLoad; (0x105630e08)
		- (void) viewDidAppear:(BOOL)arg1; (0x105630e34)
		- (void) traitCollectionDidChange:(id)arg1; (0x105631ab4)
		- (void) viewLayoutMarginsDidChange; (0x105631f08)
		- (void) viewSafeAreaInsetsDidChange; (0x105631f34)
		- (void) scrollViewDidScroll:(id)arg1; (0x105632320)
		- (void) scrollViewWillEndDragging:(id)arg1 withVelocity:(struct CGPoint)arg2 targetContentOffset:(struct CGPoint*)arg3; (0x105632370)
		- (id) initWithNibName:(id)arg1 bundle:(id)arg2; (0x10563cedc)
		- (void) .cxx_destruct; (0x10562dd5c)
```

Next I inspected the DataSource
```
(lldb) po [@"" __methodDescriptionForClass:[_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource class]]

in _TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource:
	Instance Methods:
		- (BOOL) collectionView:(id)arg1 canMoveItemAtIndexPath:(id)arg2; (0x101c8bce0)
		- (void) collectionView:(id)arg1 moveItemAtIndexPath:(id)arg2 toIndexPath:(id)arg3; (0x101c8bec8)
		- (void) .cxx_destruct; (0x101c8c0f0)
```

I put another breakpoint at `moveItemAtIndexPath` in `_TtCC16MusicApplication29NowPlayingQueueViewControllerP33_7C0FB455470DF139DBFE3B6FE6F50CE610DataSource` and saw that this method gets called before 


image lookup -a 0x00000001034a3ec8