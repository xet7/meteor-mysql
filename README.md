# nodets:mysql
## This is a meteor package which brings strong and easy way in order to write apps using Mysql.
#### [node-mysql-wrapper for nodejs](https://github.com/nodets/node-mysql-wrapper/blob/master/README.md)

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/nodets/node-mysql-wrapper?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)


## Table of Contents


- [Install](#install)
- [Introduction](#introduction)
- [Contributors](#contributors)
- [Community](#community)
- [Establishing connections](#establishing-connections)
- [Connection options](#connection-options)
- [Terminating connections](#terminating-connections)
- [Tables](#tables)
- [Performing queries](#performing-queries)
- [Running tests](#running-tests)
- [Full documentation](#full-documentation)

## Install

```sh
$ meteor add nodets:mysql
```

## Introduction

This is a meteor wrapper for mysql driver. It contains mysql driver and mysql collections instead of Mongo.Collection.

Here is an example on how to use it:

```js
//Your client side DOES NOT REQUIRE ANY CHANGES!!!
 if (Meteor.isClient) {

     Posts = new Mongo.Collection("postsCollection");
     Meteor.subscribe('allPosts')
 }

if(Meteor.isServer){
	Meteor.startup(function() {

	//Start of changes

	var mysqlStringConnection = 	"mysql://yourusername:yourpassword@127.0.0.1/yourdatabase? debug=false&charset=utf8";

	var db = Mysql.connect(mysqlStringConnection);
	//"posts" -> a table name inside your database.
	Posts = db.meteorCollection("posts", "postsCollection");

	//End of changes, that's it!

//publish the collection as you used to do with Mongo.Collection
	 Meteor.publish("allPosts", function(){
                return Posts.find();
     });

	});
}

```

From this example, you learn the following:

* No changes needed to client side if you are converting your app from Mongo.
* At server side you need only 2 lines of code, the Mysql.connect and the db.meteorCollection.

## Contributors

Thanks goes to the people who have contributed code to this module, see the
[GitHub Contributors page][].

[GitHub Contributors page]: https://github.com/nodets/node-mysql-wrapper/graphs/contributors



## Community

If you'd like to discuss this module, or ask questions about it, please use one
of the following:

* **Chat**: https://gitter.im/nodets/node-mysql-wrapper
* **Mailing list**: https://groups.google.com/forum/#!forum/node-mysql-wrapper
* **IRC channel**: http://irc.lc/freenode/node-mysql-wrapper/


## Establishing connections

The recommended way to establish a connection is this:

```js
var db = Mysql.connect("mysql://user:pass@127.0.0.1/databaseName?debug=false&charset=utf8");
```

However, a connection can also be implicitly established by wrapping connection properties:

```js
var connectionSettings = {
  host     : 'localhost',
  user     : 'me',
  password : 'secret',
  database : 'my_db'
};

var db = Mysql.connect(connectionSettings);
```

Depending on how you like to code, either method may be
appropriate. But in order to works .

## Connection options

Read them at the [node-mysql module](https://github.com/felixge/node-mysql/#connection-options) documentation



## Terminating connections

There are two ways to end a connection. Terminating a connection gracefully is
done by calling the `end()` method:

```js
db.end(function(err) {
  // The connection, table events and queries are terminated now
});
```

This will make sure all previously enqueued queries are still before sending a
`COM_QUIT` packet to the MySQL server. If a fatal error occurs before the
`COM_QUIT` packet can be sent, an `err` argument will be provided to the
callback, but the connection will be terminated regardless of that.

An alternative way to end the connection is to call the `destroy()` method.
This will cause an immediate termination of the underlying socket.
Additionally `destroy()` guarantees that no more events or callbacks will be
triggered for the connection.

```js
db.destroy();
```

Unlike `end()` the `destroy()` method does not take a callback argument.


## Tables

### Manual select which tables you want to use. (default all)

```js
 Mysql.connect('mysqlConnectionString','table1','table2','table3');
```

### Getting a table object
```js
//all code you will see bellow goes after Mysql.connect function.
var usersTable = db.table("users"); //yes, just this :)
console.log('available columns: '+ usersTable.columns);
console.log('primary key column name: '+ usersTable.primaryKey);
console.log('find, findById, findAll, save and remove methods can be called from this table too instead of the meteorCollection, but this is not necessary');


```
## Performing queries

### Collection queries for server and client side

```js
//Surpise!!! Performing simple queries like you used with  Mongo.Collection :)

var story= Stories.findOne({storyId:5}); //Ojbect
var storiesFromAuthor1 = Stories.find({authorId:1}); //Array

```
###Collection queries to the mysql database on server side only.

**An 'advanced  find' method**. Find all users where years_old = 22, find the user's info, find user's comments, the comment's likes and users who liked them.
```js
//This library also offers you a lot of help to write advanced mysql queries without need to know any sql commands.


var criteria = db.criteriaFor("users")
.where("yearsOld").gt(18)
.exclude("password","createdDate") //or .except(...columns). Removes that column(s) from the select query.
.joinAs("info","userInfos","userId").at("info").limit(1)
//with.at('tableOrPropertyName') we are going and passing criterias inside the info property
//this will pass the the result as object not as array, because of limit(1)
.parent() // or .original() here will be redirect to parent object, ( user(s) table) to continue our query...
//original() goes to the first-original-primary table, parent() goes to the parent table, you can have unlimited .at('joinedTableOrProperty') functions.
.join("comments","userId")
.at("comments")
.orderBy("commentId",true) //true if you want desceding ( ORDER BY COLUMN_KEY DESC )
.joinAs("likes","commentLikes","commentId")
.at("likes")
.joinAs("likers","users","userId").build();

Users = db.meteorCollection("users","usersCollection",criteria);
//criteria object offers you executing select queries to the mysql database.
//Use it if you need to return only specific columns or joined tables.

console.dir(Users.find());
```

**Insert method**,  Returns a single object, also updates the variable you pass into.
```js

var newUser = { username: 'a new user', yearsOld: 23, mail: 'newmail@mail.com' };

//Like you used with Mongo.Collection

var newUserId = Users.insert(newUser);

```

**Remove method .1**

```js
//remove/delete all rows from users table where years_old = 22
Users.remove({yearsOld:22});

```

**Remove method .2** remove by id also

```js
//remove/delete a single row by its primary key
Users.remove(4); //User with user_id (userId)  = 4 removed.
```

## Running tests


### Import this database example to your local server, and have fan!
```sql

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for comments
-- ----------------------------
DROP TABLE IF EXISTS `comments`;
CREATE TABLE `comments` (
  `comment_id` int(11) NOT NULL AUTO_INCREMENT,
  `content` varchar(255) DEFAULT NULL,
  `user_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`comment_id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of comments
-- ----------------------------
INSERT INTO `comments` VALUES ('1', 'dsadsadsa', '18');
INSERT INTO `comments` VALUES ('2', 'wqewqewqeq', '23');
INSERT INTO `comments` VALUES ('3', 'cxxzczxczcz', '22');
INSERT INTO `comments` VALUES ('4', 'e comment belongs to 23 usersa', '23');
INSERT INTO `comments` VALUES ('5', 'dsadsadsadsa 23', '23');
INSERT INTO `comments` VALUES ('6', 'ewqewqewqeq 23dsad', '23');
INSERT INTO `comments` VALUES ('7', 'ewqewqewqeq 23', '23');
INSERT INTO `comments` VALUES ('8', 'vcxvcxvcvcx 22 ', '22');
INSERT INTO `comments` VALUES ('9', 'dsadsadsadsa 22', '22');
INSERT INTO `comments` VALUES ('10', 'vcbvcbvcbvcbvcbcv 18', '18');
INSERT INTO `comments` VALUES ('11', 'nbvnbvnbvnbvnbvnbvnbv 22', '22');
INSERT INTO `comments` VALUES ('12', 'mnbmnbmnbmnbmnmbn 18', '18');

-- ----------------------------
-- Table structure for comment_likes
-- ----------------------------
DROP TABLE IF EXISTS `comment_likes`;
CREATE TABLE `comment_likes` (
  `comment_like_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `comment_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`comment_like_id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of comment_likes
-- ----------------------------
INSERT INTO `comment_likes` VALUES ('1', '18', '1');
INSERT INTO `comment_likes` VALUES ('3', '18', '2');
INSERT INTO `comment_likes` VALUES ('4', '12', '1');
INSERT INTO `comment_likes` VALUES ('5', '16', '3');
INSERT INTO `comment_likes` VALUES ('6', '18', '4');
INSERT INTO `comment_likes` VALUES ('7', '16', '4');
INSERT INTO `comment_likes` VALUES ('8', '16', '3');
INSERT INTO `comment_likes` VALUES ('9', '18', '3');

-- ----------------------------
-- Table structure for stories
-- ----------------------------
DROP TABLE IF EXISTS `stories`;
CREATE TABLE `stories` (
  `story_id` int(11) NOT NULL AUTO_INCREMENT,
  `content` text,
  `title` varchar(255) DEFAULT NULL,
  `author_id` int(11) DEFAULT NULL,
  `created_date` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `cover_photo` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`story_id`)
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of stories
-- ----------------------------
INSERT INTO `stories` VALUES ('1', 'Too many of my first drafts have died an early death because of that sentence.\r\nThis is an excuse that neatly covers failure for just about any work. It’s so easy to justify. But just because an excuse is justifiable doesn’t mean it’s worthwhile. Because I can always imagine a time when I’ll feel more refreshed and more energized, “I’m too tired” can ruin any chance I have to make something.\r\nWhat to do, then? Run a few laps? Get more sleep? Drink five Red Bulls? (The last one only works for my developer friends — don’t try it).\r\nHalf of the real problem of exhaustion comes from distraction. The problem is not being too tired. The problem is having a divided mind.\r\nMost of us probably have a dozen different projects going on at the same time — or worse, left ignored. This saps our confidence that we can do new work well (or completely). It keeps us from reaching the creative focus needed to get into the flow states that make work fun.\r\nThe solution I’ve found? If I want real momentum in starting something, I have to finish something. I can only start from where I left off, so I begin by fulfilling the goals or obligations I’ve left halfway to completion.\r\nAs simple as this solution sounds, getting the mental space to start something new is rarely as easy as checking off the last item on my to-do list. I have to ask myself a few questions first:\r\nWhere does this work fit into my priorities? — Completing old work should remove a stress point or obligation, or it should make starting new work simpler.\r\nHow long will it take me to complete this? — Since my main goal is to begin something new, I want to tackle something I can finish in a reasonable amount of time.\r\nWhat mental roadblocks are holding me back from finishing this? — Odds are, they’re the same ones — analysis paralysis, fear of embarrassment, fear of negative feedback — that will do even more to hold me back from starting something.\r\nWhen I have the answers for these questions, I know what to take on first. That takes some of the uncertainty out of sitting down to start a work session.\r\nMore importantly, once I have the forward momentum of having done something important, starting something new doesn’t seem nearly so hard. And when I’ve done something really cool or worth sharing, I don’t feel nearly as tired.\r\nWhat excuses are holding you back from doing great work? Don’t just procrastinate or force yourself to run up against walls — rethink the source of your problem with the work. The solution may be counterintuitive: like me, maybe you need to finish things before you can start them.', 'You’re Not Too Tired To Create. You’re Too Distracted.', '16', '2015-09-30 18:58:08', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*6dr_RzMJZge2fveF2ZI6TA.jpeg');
INSERT INTO `stories` VALUES ('2', 'There is a good chance you’ll be able to put at least one of these learning tools to good use and come out as a better person than you were last year. These are some of the best websites that will make you smarter every day.\r\nBBC — Future — Making you smarter, every day.\r\n2. 99U (YouTube) — Actionable insights on productivity, organization, and leadership to help creative people push ideas forward.\r\n3. Youtube EDU — The education videos that don’t have cute cats in boxes — but they do unlock knowledge.\r\n4. WikiWand — A slick new interface for Wikipedia.\r\n5. The long read (The Guardian) — In-depth reporting, essays and profiles.\r\n6. TED — Great videos to open your mind on almost every topic.\r\n7. iTunes U — Learning on the go, from some of the world’s top universities.\r\n8. InsightfulQuestions (subreddit) — Intellectual discussions that are not necessarily genre-specific.\r\n9. Cerego — Cerego helps you build personalized study plans based on your strengths and weaknesses to retain knowledge.\r\n10. University of the People — Tuition-free online university that offers higher education in multiple course streams.\r\n11. OpenSesame — Marketplace for online training, now with 22,000+ courses.\r\n12. CreativeLive — Take free creative classes from the world’s top experts.\r\n13. Coursera — Partnering with some of the top U.S. universities, Coursera offers massive open online courses for free.\r\n14. University of reddit — the product of free intellectualism and is a haven for the sharing of knowledge.\r\n15. Quora — You ask, the net discusses — with top experts and fascinating back and forth on everything.\r\n16. Digital Photography School — Read through this goldmine of articles to improve your photography skills.\r\n17. Umano -Explore the largest collection of audio articles powered by real people. Dropbox has acquired Umano. Brain Pickings is a great replacement for 17.\r\nBrain Pickings — Insightful long form posts on life, art, science, design, history, philosophy, and more.\r\n18. Peer 2 Peer University or P2PU, is an open educational project that helps you learn at your own pace.\r\n19. MIT Open CourseWare is a catalog of free online courses and learning resources offered by MIT.\r\n20. Gibbon — This is the ultimate playlist for learning.\r\n21. Investopedia — Learn everything you need to know about the world of investing, markets, and personal finance.\r\n22. Udacity offers interactive online classes and courses in higher education.\r\n23. Mozilla Developer Network offers detailed documentation and learning resources for web developers.\r\n24. Future learn — enjoy free online courses from top universities and specialist organisations.\r\n25. Google Scholar — provides a search of scholarly literature across many disciplines and sources, including theses, books, abstracts and articles.\r\n26. Brain Pump — A place to learn something new everyday.\r\n27. Mental Floss — Test your knowledge with amazing and interesting facts, trivia, quizzes, and brain teaser games.\r\n28. Learnist — Learn from expertly curated web, print and video content.\r\n29. DataCamp — Online R tutorials and data science courses.\r\n30. edX — Take online courses from the world’s best universities.\r\n31. Highbrow — Get bite-sized daily courses to your inbox.\r\n32. Coursmos — Take a micro-course anytime you want, on any device.\r\n33. Platzi — Live streaming classes on design, marketing and code.', '33 Websites That Will Make You a Genius', '18', '2015-09-30 18:58:20', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*Gb81VqJsnsmYAuosU4Wf6g.jpeg');
INSERT INTO `stories` VALUES ('3', 'I’m posting more of these over at InstaChaaz if you’re into graphically represented nonsense.\r\nIn the meantime, hit that recommend button, or tweet it, or whatever you like really…', 'Eight graphs to prove it’s not just you…', '19', '2015-09-30 18:58:29', 'https://d262ilb51hltx0.cloudfront.net/max/239/1*egJz27B013_QE8BAcyVTFg.png');
INSERT INTO `stories` VALUES ('4', 'July 2014, Alexis Madrigal reported for NPR on the strange case of Pinterest. The visual bookmarking app “is mostly known as a place people go to find things to buy or make,” he wrote, and — unusually for a high-flying tech company — it had first gained traction among “young women away from the coasts.” But to underestimate it would be a mistake: as “a repository of things that people would like to have or do,” he continued, Pinterest constitutes “a database of intentions.” And assuming that intentions have power and value, it stands to reason that the places where they accumulate do, too.\r\nIn presenting Pinterest as a database of intentions, Madrigal reopened a conversation started by John Battelle over a decade earlier, in 2003. At the time, Battelle had Google on the brain; the not-yet-giant was less than a year away from its IPO, and Battelle was in the thick of researching a book about its rise. “The Database of Intentions is simply this,” Battelle wrote:\r\nThe aggregate results of every search ever entered, every result list ever tendered, and every path taken as a result.\r\nIn 2010, Battelle expanded the notion. “My mistake in 2003 was to assume that the entire Database of Intentions was created through our interactions with traditional web search,” he acknowledged. “I no longer believe this to be true.” In place of this narrow definition, he perceived an ever-evolving field of signals — together constituting “no less than the sum of our economic and cultural potential.”\r\nIn his expansion, Battelle noted first four, then five types of signals:\r\nThe Query — what I want\r\nThe Social Graph — who I am, who I know\r\nThe Status Update — what I’m doing, what’s happening\r\nThe Check-In — where I am\r\nThe Purchase — what I buy\r\nBattelle freely acknowledged that his taxonomy of signals was incomplete; he closed his March 2010 update, “The Database of Intentions Is Far Larger Than I Thought,” by affirming that he was “on the lookout for new Signals,” and “quite certain we’re not nearly finished creating them.”\r\nThat very same month, a small startup set off on a path that would ultimately prove Battelle’s prediction right. It was in March 2010 that Pinterest launched in closed beta, strengthening a signal that had swirled for years without breaking through: the bookmark.\r\nThe Bookmark represents what we wish for. It’s the earliest indicator of intention, and the most vulnerable; by definition, the act of saving something for later means that whatever we hope for hasn’t happened yet. Bookmarks are placeholders for the future. By thumbing through them, we can start to see what might happen next.', 'Save for Later', '20', '2015-09-30 18:58:35', 'https://d262ilb51hltx0.cloudfront.net/max/462/1*1o3X33zKak3GjEfHl0G0YQ.jpeg');
INSERT INTO `stories` VALUES ('5', ' a particular look that people have when they come into my office to tell me they’re pregnant (and as an editor with a mostly young, female staff, this happens a lot). The expression is one part joy, and one part anxiety — and while I always hope they’ll leave my office feeling all of the former and none of the latter, I know that particular cocktail of personal-life happiness and telling-the-boss dread all too well, because I lived it.\r\nWhen my husband and I first started talking about having a baby, I was 33, and editor of a magazine I felt I had well in hand. It would be hard to pull off a maternity leave, but not TOO hard. Then I got promoted, to a bigger job with bigger stakes. We decided to wait a bit. (Some friends tsk-tsked; “don’t put it off — no job is worth that,” snapped one mentor.) But about ten months in, the time seemed right. I was 34; work was demanding, but not crushingly so; my husband, a film producer, was between projects and could be at home for much of a baby’s early months. And my pregnancy was blessedly easy. As I jetted back and forth to the European fashion collections in those early months, covering my not-yet-announced belly and grabbing catnaps between shows, I felt my life was firing on all cylinders.\r\nMaybe it was pregnancy hormones, but this was a cinch! I could do this! Thank God for tent-dresses!\r\nUntil I started thinking about telling my boss. Suddenly everything seemed more fraught. Would he worry about my pending absence? Would he criticize my timing? And what was I thinking with this timing, anyway? Most of my friends in similar jobs had had their babies long before becoming editors-in-chief — wait, why hadn’t I thought of that? By the time I pressed the 11th-floor elevator button to go see him to break the news, I was in a full body sweat, and Bernard Herrmann’s music for the shower scene in Psycho was playing in the back of my mind. (I was also seriously straining at the seams of a denim Gucci skirt that hadn’t fit me for at least a month. I had been putting off this meeting for a while.) My husband, mystified by why I was worried, had offered some kitchen-table advice that morning — “just be businesslike, upbeat and casual” — and I clung to that mantra as I marched down the hushed hallway. Businesslike, upbeat, and casual. Businesslike, upbeat, and casual.\r\nIn the end, those three words got me through. I blurted my news, my boss beamed with authentic happiness (and asked me whether I already had kids — oh, I guess the details of my personal life weren’t quite as vital to him as to me), and then we went on to the rest of the meeting’s business. The volume lowered on the Psycho soundtrack. If I was going to act confident and unconcerned about my ability to do two things well, then, it seemed, everyone else would follow along.\r\nWhich is, of course, how it should be.\r\nPregnancy, no matter when you decide to go about it, is as normal and common as a human activity can be; as epic as my pregnancy seemed to me, I was doing nothing particularly unusual in having a baby, and it was reassuring for me to be reminded of that.\r\n(On a similar note, I once delivered a calf on a dairy farm — long story — and during my own labor found it strangely comforting to remember how unruffled the mother cow had been by the whole process. Females get pregnant. We have babies. That cow was happily munching hay five minutes after delivery and I’m pretty sure she could have edited a solid issue of Bovine Monthly with no problem if she’d had to.) Parenthood is full of challenges, but just announcing it should not be one of them.\r\nFor too many women, of course, that moment is much more difficult than it was for me. Not every boss is as enlightened and unworried as mine was; simply chanting “businesslike, upbeat, casual” won’t get you too far if the head honcho sees your leave as annoying, or the company won’t grant you a paid one (yes, that’s still legal!), or your superior secretly doubts that you can do what working fathers have done for millennia and procreate while still being awesome on the job. I had it lucky.\r\nAll women deserve that kind of straightforward encouragement, and I think not apologizing for wanting to do the thing that, you know, our entire species depends on is a good start — especially while we wait for our legislators and employers to catch up. The moment you decide to have a baby is both extraordinary (to you) and ordinary (to the universe), and you will do it, with its ups, downs, and unexpected roller-coaster turns, as women, men, cows, cats, and every other living thing has before you.\r\nBusinesslike, upbeat, casual, wonderful.\r\nNow tell me: If you’ve made the decision to become a parent, when, and how, did you do it?', 'Women Work. We Have Babies. Get Over It.', '22', '2015-09-30 18:58:08', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*9mM8ks_px9cSKtS_S76rFA.jpeg');
INSERT INTO `stories` VALUES ('6', 'many years ago, I was sitting on an airplane chatting with an agreeable man in the next seat. He worked at NASA, and his job was to make sure that nothing that left earth on a spacecraft would contaminate the environment on other planets. He gave me his card, and it had the best job title I’ve ever seen: Planetary Protection Officer.\r\nI thought of him again this morning when two remarkable stories criss-crossed in the news: the discovery of liquid water on Mars, and Shell’s decision to back down from drilling in the Arctic.\r\nThe first was a great scientific achievement, as spectrographs on NASA’s Mars Reconnaissance Orbiter were able to show through the analysis of chemical signatures that intermittent dark streaks that appear and then fade on Martian canyon walls had to be water.\r\nThat’s amazing — it certainly heightens the chance that there could be microbial life on the red planet.\r\nBut almost as amazing was to read the details of the story and learn that my seat-mate’s successor as NASA’s planetary protection officer, a woman named Catharine Conley, was not letting the Mars Rover anywhere near the streaks, though some of them were within driving distance.\r\nThe vehicle hadn’t been fully sterilized before it left earth; therefore at least for now the streaks were off limits. We’re taking enormous care to make sure they stay pristine\r\nMeanwhile, back on our own planet, Shell announced that it was pulling the plug on efforts to drill in the Arctic “for the foreseeable future.” The official reason was that they hadn’t found as much oil as they’d hoped for. The unofficial reason, as sources in Shell made clear to reporters, was the brand damage they’d suffered — and rightly so.\r\nThis was a company prepared until this morning to take advantage of the degree to which the planet had already warmed by drilling the thawed Arctic for yet more oil to run up the temperature some more. Just think about that for a moment.\r\n\r\nEnough activists thought about that to make Shell’s life impossible. The company can greenwash a lot — they’re currently trying to rehabilitate their image so they can ‘advise’ European governments on the upcoming climate talks in Paris — but they couldn’t greenwash this. As The Guardian reported this morning, “company sources also accept that Arctic oil polarized debate in a way that damaged the firm. “We were acutely aware of the reputational element to this programme,” one said.\r\nCombined with the ongoing halt to the Keystone pipeline, and the recent end to plans for the world’s largest coal mine in Australia, it means activists have helped to begin defusing three of the planet’s dozen or so largest ‘carbon bombs.’ And Shell’s capitulation will make the next fights easier.\r\nIt shouldn’t have to be this way. In a rational world governments would be working overtime to shut off the flow of carbon to the atmosphere — instead it was Barack Obama who gave Shell the green light to go north.\r\nSo for now, only other planets have official protection against pollution.', 'How to Protect a Planet', '23', '2015-09-30 18:58:20', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*bUc6fciBgRvgS4_TKQW2mQ.jpeg');
INSERT INTO `stories` VALUES ('7', 'They say that a rising tide lifts all boats. Well, that’s a bunch of bullshit.\r\nWe’ve got a rising tide in San Francisco. The tech industry has brought thousands of people and billions of dollars to these forty nine square miles. Traffic is thick with Teslas. The WiFi is stuffed with startups. Times are good.\r\nExcept where they’re not.\r\nA few short blocks away from that tide where companies like Twitter are revolutionizing the information age, the kids in the Tenderloin are walking the same bleak blocks many of their parents walked when they were kids.\r\nLike so many of you, I’ve benefited greatly by being at the right place at the right time with the right opportunities. But I’ve also seen the kids who are not benefiting — even a little — from Internet gold rush.\r\nThat’s not right. But it’s not going to change by itself. It’s going to take great organizations like 826 Valencia. I’m on the board, and I can vouch that the folks there know how to lift kids up, to inspire them, to teach them to write well, to feed their curiosity, to let them dream. They’ve been doing it for years, ever since Dave Eggers and others started the organization.\r\nAnd now we’re bringing 826 to the Tenderloin. We took over a building once occupied by a notorious liquor store on a crime-ridden corner, and it will be transformed into a place of creativity, hope, and fun.\r\nThis is what the tech community owes to the kids who live in the shadows of our high rises. Supporting a center like this — and an organization with a proven track record — is the right thing to do.\r\nThe tide won’t lift these kids. But with your help, 826 Valencia will.\r\nWhat do we need from you? One thing. It will take about three seconds.\r\nJust vote for 826 Valencia in the Google Impact Challenge and $250k will be added to the money we have to build the new center and start our programming.\r\n\r\nA big tech company giving money to the kids in the Tenderloin. That’s a tide I can ride.', 'A Few Blocks From Medium…', '24', '2015-09-30 18:58:29', 'https://d262ilb51hltx0.cloudfront.net/max/359/1*DuZrdiG0AF_s1zNkoG78dw.jpeg');
INSERT INTO `stories` VALUES ('8', 'Flux is both one of the most popular and one of the least understood topics in current web development. This guide is an attempt to explain it in a way everyone can understand.\r\nThe problem\r\nFirst, I should explain the basic problem that Flux solves. Flux is a pattern for handling data in your application. Flux and React grew up together at Facebook. Many people use them together, though you can use them independently. They were developed to address a particular set of problems that Facebook was seeing.\r\n\r\nOne really well known example of this set of problems was the notification bug. When you logged in to Facebook, you would see a notification over the messages icon. When you clicked on the messages icon, though, there would be no new message. The notification would go away. Then, a few minutes later after a few interactions with the site, the notification would come back. You\'d click on the messages icon again… still no new messages. It would just keep going back-and-forth in this cycle.\r\n\r\nIt wasn’t just a cycle for the user on the site. There was also a cycle going on for the team at Facebook. They would fix this bug and everything would be fine for a while and then the bug would be back. It would go back-and-forth between being resolved and being an issue again.\r\nSo Facebook was looking for a way to get out of this cycle. They didn’t want to just fix it once. They wanted to make the system predictable so they could ensure that this problem wouldn’t keep resurfacing.\r\nThe underlying problem\r\nThe underlying problem that they identified was the way that the data flowed through the application.\r\nNote: this is what I’ve gleaned from simplified versions that they’ve shared in talks. I’m sure the actual architecture looked different.\r\n\r\nModels pass data to the view layer.\r\nThey had models which held the data and would pass data to the view layer to render the data.\r\nBecause user interaction happened through the views, the views sometimes needed to update models based on user input. And sometimes models needed to update other models.\r\nOn top of that, sometimes these actions would trigger a cascade of other changes. I envision this as an edge-of-your-seat game of Pong — it’s hard to know where the ball is going to land (or fall off the screen).\r\n\r\nViews update models. Models update other models. This starts to look like a really edge-of-your-seat game of Pong.\r\nThrow in the fact that these changes could be happening asynchronously. One change could trigger multiple other changes. I imagine this as throwing a whole bag of ping-pong balls into your Pong game, with them flying all over the place and crossing paths.\r\nAll in all, it makes for a hard to debug data flow.\r\nThe solution: unidirectional data flow\r\nSo Facebook decided to try a different kind of architecture, where the data flows in one direction — only one direction — and when you need to insert new data, the flow starts all over again at the beginning. They called their architecture Flux.', 'A cartoon guide to Flux', '28', '2015-09-30 18:58:35', 'https://d262ilb51hltx0.cloudfront.net/max/600/1*epbnjfECHp2te0Fx4lJhgA.png');
INSERT INTO `stories` VALUES ('9', 'When I was 17 years old, I used to work and study for about 20 hours a day. I went to school, did my homework during breaks and managed a not-for-profit organization at night. At that time, working hard landed me countless national campaigns, opportunities to work with A-list organizations and a successful career. As I got older, I started thinking differently. I realized that working harder is not always the right path to success. Sometimes, working less can actually produce better results.\r\nConsider a small business owner, who works non-stop. However, working hard won’t help him compete with his multi-million competitors. Time is a limited commodity. An entrepreneur can work 24 hours a day and 7 days a week (the most amount of time anyone can work, really). His or her competitor can always spend more money, build a bigger team and spend a lot more time on the same project. Then why have small startups accomplished things that larger corporations couldn’t? Facebook bought Instagram, a 13-employee company for a billion dollars. Snapchat, a young startup with 30 employees is turning down offers from tech giants Facebook and Google. Part of their successes were based on luck — the rest is based on efficiency.\r\nThe key to success is not hard working but smart working.\r\nThere’s a notable distinction between being busy and being productive. Being busy doesn’t necessarily mean you’re being productive. Being productive is less about time management and more on managing your energy. It is the business of life. We need to learn how to spend the least amount of energy to get the most benefits. I am so lucky to work with an amazing team here at Filemobile. Everyone always challenges me and helps me sort my priorities to become more productive. I learned to reduce my work week from 80 hours to 40 hours, and get a lot more work done in the process. In other words, less is more.\r\nHere are 7 I things I stopped doing to become more productive.\r\n1. Stop working overtime and increase your productivity\r\nHave you ever wondered where the 40-hour work week came from? In 1926, Henry Ford, American industrialist and founder of Ford Motor Company, conducted experiments with interesting results: when you decrease your daily working hours from 10 to 8, and shorten the work week from 6 days to 5, your productivity increases.', '7 Things You Need To Stop Doing To Be More Productive, Backed By Science', '31', '2015-09-30 18:58:35', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*N3IF6fxdWCq_y9-36FgiSQ.jpeg');
INSERT INTO `stories` VALUES ('10', 'This project was a collaboration between 18F and the U.S. Digital Service. The team was lead by Mollie Ruskin (USDS) and Julia Elman (18F) and made up of designers and developers in both groups, including Maya Benari (18F), Carolyn Dew (18F), Victor Garcia (USDS), Angel Kittiyachavalit (USDS), Colin MacArthur (18F), and Marco Segreto (18F).\r\nMeet Joanne — she’s a young Army Veteran who is looking to make use of her GI Bill Benefits and apply for federal student loans to attend college.\r\n\r\nAs she tries to access federal programs which will allow her to afford college, Joanne navigates multiple agency websites. She finds dozens; they all seem relevant to what she’s looking for.\r\nNaturally, Joanne is confused. Are these programs related to each other? Are they even all a part of the federal government? Are any of these a scam? When she tries to access the sites during her morning commute, she finds half of them are impossible to use on her phone. She’s overwhelmed by how hard these tools are to use; she feels frustrated and isolated, and worries she might miss opportunities she’s eligible for.', 'Introducing:\r\nU.S. Web Design Standards', '16', '2015-09-30 18:58:08', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*dNOWb2gHTLF8jNHKbXHkTA.png');
INSERT INTO `stories` VALUES ('11', 'It has been seriously weird to watch this article spread like crazy (reaching most-recommended on Medium!) as a writer who normally writes, yanno, ACTUAL articles. If you like the sentiment behind it, subscribe to get the rest of my writing at SorryForMarketing.com. Quick intro: I’m Jay Acunzo, VP at NextView (seed VC), ex-Googler, and host of Traction, a 5-star startup podcast where founders of companies like LinkedIn, DraftKings, Mattermark and more share creative & clever things they did to make early progress.', 'The One Secret Thing All Successful People Do', '16', '2015-09-30 18:58:20', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*QPPAZCZBmUHrP8-pINd-gA.jpeg');
INSERT INTO `stories` VALUES ('12', 'In the past 3 years, I have already run my business from Lebanon, Indonesia, Morocco, Iran, Kenya, and South Africa. Always on the hunt after a stable Wi-Fi connection, but free of fixed working hours.\r\nMy office can be my bed as well as my favorite local coffee shop. My only tools are a laptop, a smartphone, and a pair of headphones to isolate myself from the noises of the outer world. Needless to say, I’m 100% dependent on technology to be able to work from anywhere the word.\r\nTechnology sometimes fails me. There’s no Internet. My computer freezes up. The phone is dead.\r\nBut most of the time, it is technology what makes it possible to keep my dream of time and location freedom alive. Both quality hardware and handy software. That’s why I’ve selected a list of really useful apps that will help you stay connected, productive, and above all relaxed.\r\n\r\nDOCUMENTS\r\n\r\n\r\n1. Docady\r\nSnap a photo of all the important documents you don’t want to carry around. The app will help you arrange the paperwork into relevant categories and safely store everything you might need on your travels.\r\n\r\n2. PandaDoc\r\nThere’s no need to send your proposals or contracts via snail mail. You can easily create, share, and sign documents with legally binding electronic signature software.\r\n\r\n3. Docracy\r\nThere’s also no rush to look for a lawyer every time you need a simple contract. Use this amazing open database of legal documents instead.\r\n\r\nSECURITY\r\n\r\n\r\n4. CrashPlan\r\nProtect your data and back up all your valuable files (music, photos, documents) to an external drive or cloud. You can access them from anywhere, at any time.\r\n\r\n5. Hola\r\nIn some countries, your popular websites (e.g. Facebook, Twitter, YouTube…) may be blocked. For such cases, it’s good to have a solid VPN installed before you cross the borders, so you can browse the Internet without censorship.\r\n\r\nPRODUCTIVITY\r\n\r\n\r\n6. Sunrise\r\nPlan your days with a calendar that connects to your favorite apps, including Facebook, Evernote, or Trello.\r\n\r\n7. Trello\r\nKeep track of your daily tasks and collaborate with your team on any project with this free and simple board-like app. No more endless email threads or missed deadlines.\r\n\r\n8. Wunderlist\r\nOrganize your to-dos, set due dates, and reminders. You can also share your lists with other people and possibly assign the tasks to someone else.\r\n\r\n9. Solo\r\nIf you work as a freelancer, most project management apps probably don’t address your specific needs. Solo will not only help you get stuff done, it will also assist with monitoring your performance, tracking your time and expenses, generating invoices, or managing clients.\r\n\r\n10. WudaTime\r\nEasily track the time you spend on individual tasks with the help of this free browser-based tool.\r\n\r\n11. Gorgias\r\nSave some minutes every time you send an email. This app will help you create nifty templates and shortcuts so you fly through your inbox super fast.\r\n\r\nDISCIPLINE\r\n\r\n\r\n12. Momentum\r\nSelf-discipline is the main challenge when being your own boss. This app will help you create new habits, track your daily progress, and achieve your goals.\r\n\r\n13. Freedom\r\nBlock distracting websites and apps to skyrocket your productivity.\r\n\r\nCOMMUNICATION\r\n\r\n\r\n14. Pie\r\nStay connected with your remote coworkers via an effortless team chat app. Share messages, images, GIFs, videos, music, and files. Find anything with a top-notch search function and sync everything between all devices.\r\n\r\n15. Speak\r\nThose who prefer to speak face-to-face can complement chatting with team video calls.\r\n\r\nPEOPLE\r\n\r\n\r\n16. Clarity\r\nSchedule a call with an experienced entrepreneur and get expert advice on your business no matter where in the world you are.\r\n\r\n17. Meetup\r\nMeet new people at events organized by local communities all over the globe.\r\n\r\n18. Idealist\r\nIf you wish to combine work, travel, and making the world a better place, check one of many volunteer opportunities here.\r\n\r\nTRAVEL\r\n\r\n\r\n19. Entrain\r\nThose who often suffer from jet lags will make good use of this free app. It will tell you when to seek dark or light to adjust faster to new time zones.\r\n\r\n20. PackPoint\r\nThis smart packing list app will check the weather, note the length of your stay, and consider the purpose of your trip, so you don’t forget anything important.\r\n\r\nAND MORE\r\n\r\n\r\n21. Dollarbird\r\nControl your budget on your smartphone. See how much you’ve spent, plan your future expenditures, and observe your spending habits.\r\n\r\n22. Silvrback\r\nDon’t waste your time setting up a new website. Capture your adventures in a lean, distraction-free blog.\r\n\r\n23. Canva\r\nPresentations, infographics, blog post images, business cards, web banners, posters, invitations… any design related stuff is a piece of cake even for non-designers with this free, drag-and-drop app.\r\n\r\n24. CodersClan\r\nGet your smaller coding tasks done by professional coders in no time.\r\n\r\n25. SuperTasker\r\nDo you need to fix your WordPress site, create a new logo, or write a press release? All sorts of digital tasks can be delivered within hours through this website.', 'Work From Anywhere: 25 Cool Apps For Digital Nomads, Corporate Escapists, And Loony Adventurers', '19', '2015-09-30 18:58:29', 'https://d262ilb51hltx0.cloudfront.net/max/408/1*-yhO0xml2xcxXrkpnBpoeg.jpeg');
INSERT INTO `stories` VALUES ('13', 'iOS 9 is now publicly released. It’s a subtle change but the system fonts of iOS 9 are now changed to the Apple’s new San Francisco fonts, replacing the previous Helvetica Neue.Apple has been using Helvetica as the system fonts for iOS since the first iPhone, and they also switched the fonts from Lucida Grande to Helvetica for Mac OS X since 10.10 Yosemite. Why did Apple decide to ditch Helvetica, which is the most famous and loved font in the world?\r\nHelvetica is weak in small sizes\r\nIt is said that Helvetica is not suited for texts in small sizes. When the system fonts for Mac OS X Yosemite are changed to Helvetica, many designers claimed that Helvetica is not appropriate one.San Francisco is not a single font\r\nSan Francisco fonts have lots of features to be highly legible. In fact, the San Francisco fonts for Apple Watch and for iOS/Mac are not same.\r\nThe font family named “SF” is used for iOS/Mac and “SF Compact” is for Apple Watch. You can see the difference in round shaped letters like ‘o’, ‘e’. SF compact has rather flat vertical lines than that of SF.', 'The Secret of the Apple’s New San Francisco Fonts', '22', '2015-09-30 18:58:35', 'https://d262ilb51hltx0.cloudfront.net/fit/t/800/240/1*8lz2kG3qEW3R1y_K7owIKg.png');

-- ----------------------------
-- Table structure for users
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `created_date` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `years_old` int(11) DEFAULT '0',
  `avatar` varchar(255) DEFAULT NULL,
  `firstname` varchar(255) DEFAULT NULL,
  `lastname` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5961 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of users
-- ----------------------------
INSERT INTO `users` VALUES ('16', ' user 16 mail ust an email', ' username for user id 16', 'gfdgfdgfd', '2015-10-01 16:53:43', '0', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*lQ9SxlZeIUkePRdw.JPG', 'Michael ', 'Anthony');
INSERT INTO `users` VALUES ('18', 'an updated mail for user id 18 2nd time', 'an updated username for user_id 18 3rd time', 'a pass', '2015-10-01 03:16:07', '55', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*qAsYJpVR4y15iIEM.png', 'Bill ', 'McKibben');
INSERT INTO `users` VALUES ('19', 'updated19@omakis.com', 'an 19 username', 'a pass', '2015-10-01 03:16:14', '22', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/1*46PfjEw8QIoZpLy2P3SNIg.jpeg', 'Lucy', 'Bellwood');
INSERT INTO `users` VALUES ('20', 'mail20_updated@omakis.com', 'an updated20 username', 'a pass', '2015-10-01 03:16:22', '15', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*w2EL1WU52A-Ark8P.jpeg', 'Cindi', 'Leive');
INSERT INTO `users` VALUES ('22', 'mail22@omakis.com', 'a username', 'a passing', '2015-10-01 03:16:30', '22', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*AYp5fOB5FhkIwcC_.jpg', 'Sandra', 'Upson');
INSERT INTO `users` VALUES ('23', 'mailwtf@dsadsa.com', 'a username', 'pass', '2015-10-01 03:16:43', '22', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*GJgF4NfUdjPmpvfN.jpg', 'Ben', 'Panico');
INSERT INTO `users` VALUES ('24', 'mail 24', 'username 24', 'password 24', '2015-10-01 03:16:49', '23', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/1*nFh6QVVdk2orQNrz1P5aLg.png', 'Thomas', 'Oppong');
INSERT INTO `users` VALUES ('28', 'an updated username for user_id 28  or 283rd time', 'an updated x username 2nd time', 'ewqewq', '2015-10-01 03:16:56', '15', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*u67XR3Dvyk-axJZg.png', 'Chaz', 'Hutton');
INSERT INTO `users` VALUES ('31', 'an updated username for user_id 31  or 31 2nd time', 'an updated x username 1nd time', 'dsadsada', '2015-10-01 03:17:05', '0', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/1*Cy0LBwZNc0Xjffyqv_37lA.png', 'Aaron', 'Bleyaert');
INSERT INTO `users` VALUES ('5958', 'gdfgfd@hotmail.com', 'gdfgfd', 'gfdgfd', '2015-10-01 03:17:16', '0', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*mGKmWktABYWnBwl7.jpeg', 'Diana', 'Kimball');
INSERT INTO `users` VALUES ('5960', 'a@hotmail.com', 'dsadsa', 'dsadasds', '2015-10-01 03:17:20', '0', 'https://d262ilb51hltx0.cloudfront.net/fit/c/36/36/0*b5Kzlws32f2EuhGE.png', 'Dave', 'Pell');

-- ----------------------------
-- Table structure for user_infos
-- ----------------------------
DROP TABLE IF EXISTS `user_infos`;
CREATE TABLE `user_infos` (
  `user_info_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `hometown` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`user_info_id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user_infos
-- ----------------------------
INSERT INTO `user_infos` VALUES ('1', '18', 'xanthi');
INSERT INTO `user_infos` VALUES ('3', '22', 'tou 22 user hometown');
INSERT INTO `user_infos` VALUES ('4', '23', 'tou 23 user hometown');
INSERT INTO `user_infos` VALUES ('5', '24', 'xanthi');
INSERT INTO `user_infos` VALUES ('6', '24', 'allh');
INSERT INTO `user_infos` VALUES ('7', '24', 'xanthi');


```

##Full Documentation

We covered most of the common usages here, but there are more.
[You can press here to navigate to the full documentation of the node-mysql-wrapper](https://github.com/nodets/node-mysql-wrapper/blob/master/README.md) (for nodejs) (nodets:mysql for meteor)

## Licence

This project is licensed under the MIT license.

[npm-image]: https://img.shields.io/npm/v/node-mysql-wrapper.svg
[npm-url]: https://npmjs.org/package/node-mysql-wrapper
[travis-image]: https://img.shields.io/travis/nodets/node-mysql-wrapper/master.svg?label=linux
[downloads-url]: https://npmjs.org/package/node-mysql-wrapper
