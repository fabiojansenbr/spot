# Spot

Take a virtual journey around the world with Spot.

<a href="https://apps.apple.com/us/app/spot-videos/id1564675926?utm_source=github&utm_campaign=static" target="blank"><img src="https://raw.githubusercontent.com/dshukertjr/spot/main/assets/readme/download_ios.svg" alt="Download from AppStore" height="96" /></a></span><a href='https://play.google.com/store/apps/details?id=app.spotvideo&utm_source=github&utm_campaign=static&pcampaignid=pcampaignidMKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1' target="blank"><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png' height="100" /></a>

Spot is a geo-tagged video sharing app, meaning every video recorded in Spot is saved on a location. Anybody can scroll though the map and find the videos recorded in any location in the world.

DISCOVER
With Spot, you can view what is going on anywhere in the world. All videos in Spot are geo-tagged, meaning they are marked at the location they were recorded. That means if you want to take a look at what is going on in New York right now, you just go to NewYork within the app and view what people are posting. Same for LA, Paris, Tokyo or anywhere else in the world!

CREATE
You can record Spot videos to let your everyone else know what is happening wherever you are. With our intuitive video recorder, you will be able to capture the moment easily.

ENJOY
How you utilise Spot is up to you! You may be able to check out the local festival without actual going there, discover hidden local amazing restaurants, or virtually visit your favorite city in the world. When you visit amazing places, don't forget to save that memory inside Spot so that the world can know about that place!

![Spot Screenshots](https://github.com/dshukertjr/spot/blob/main/assets/readme/screenshots.png?raw=true)

---

## Figma
https://www.figma.com/file/OBSvD6eG4eDno3aQ76Ovzo/Spot?node-id=2%3A1023

---
## Supabase Database Schema

> Please note that you need enable `postgis` database extensions before run import database schema

```sql
create table if not exists public.users (
  id uuid references auth.users on delete cascade not null primary key,
  name varchar(18) not null unique,
  description varchar(320) not null,
  image_url text,
  
  constraint username_validation check (char_length(name) >= 1)
);
comment on table public.users is 'Holds all of users profile information';

alter table public.users enable row level security;
create policy "Public profiles are viewable by everyone." on public.users for select using (true);
create policy "Can insert user" on public.users for insert with check (auth.uid() = id);
create policy "Can update user" on public.users for update using (auth.uid() = id) with check (auth.uid() = id);
create policy "Can delete user" on public.users for delete using (auth.uid() = id);


create table if not exists public.videos (
    id uuid not null primary key DEFAULT uuid_generate_v4 (),
    user_id uuid references public.users on delete cascade not null,
    created_at timestamp with time zone default timezone('utc' :: text, now()) not null,
    url text not null,
    image_url text not null,
    thumbnail_url text not null,
    gif_url text not null,
    description varchar(320) not null,
    location geography(POINT) not null
);
comment on table public.videos is 'Holds all the video videos.';

alter table public.videos enable row level security;
create policy "Videos are viewable by everyone. " on public.videos for select using (true);
create policy "Can insert videos" on public.videos for insert with check (auth.uid() = user_id);
create policy "Can update videos" on public.videos for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
create policy "Can delete videos" on public.videos for delete using (auth.uid() = user_id);


create table if not exists public.comments (
    id uuid not null primary key DEFAULT uuid_generate_v4 (),
    video_id uuid references public.videos on delete cascade not null,
    user_id uuid references public.users on delete cascade not null,
    created_at timestamp with time zone default timezone('utc' :: text, now()) not null,
    text varchar(320) not null,

    constraint comment_length check (char_length(text) >= 1)
);
comment on table public.comments is 'Holds all of the comments created by the users.';

alter table public.comments enable row level security;
create policy "Comments are viewable by everyone. " on public.comments for select using (true);
create policy "Can insert comments" on public.comments for insert with check (auth.uid() = user_id);
create policy "Can update comments" on public.comments for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
create policy "Can delete comments" on public.comments for delete using (auth.uid() = user_id);


create table if not exists public.likes (
    video_id uuid references public.videos on delete cascade not null,
    user_id uuid references public.users on delete cascade not null,
    created_at timestamp with time zone default timezone('utc' :: text, now()) not null,
    PRIMARY KEY (video_id, user_id)
);
comment on table public.likes is 'Holds all of the like data created by thee users.';

alter table public.likes enable row level security;
create policy "Likes are viewable by everyone. " on public.likes for select using (true);
create policy "Users can insert their own likes." on public.likes for insert with check (auth.uid() = user_id);
create policy "Users can delete own likes." on public.likes for delete using (auth.uid() = user_id);


create table if not exists public.follow (
    following_user_id uuid references public.users on delete cascade not null,
    followed_user_id uuid references public.users on delete cascade not null,
    followed_at timestamp with time zone default timezone('utc' :: text, now()) not null,
    primary key (following_user_id, followed_user_id)
);
comment on table public.follow is 'Creates follow follower relationships.';

alter table public.follow enable row level security;
create policy "Follows are viewable by everyone. " on public.follow for select using (true);
create policy "Users can follow anyone" on public.follow for insert with check (auth.uid() = following_user_id);
create policy "Users can unfollow their follows and ssers can remove their followers" on public.follow for delete using (auth.uid() = following_user_id or auth.uid() = followed_user_id);

create table if not exists public.blocks (
    user_id uuid references public.users on delete cascade not null,
    blocked_user_id uuid references public.users on delete cascade not null,
    created_at timestamp with time zone default timezone('utc' :: text, now()) not null,
    primary key (user_id, blocked_user_id),
  
    constraint username_validation check (user_id != blocked_user_id)
);
comment on table public.blocks is 'Holds information of who is blocking who.';

alter table public.blocks enable row level security;
create policy "Users can view who they are blocking." on public.blocks for select using (auth.uid() = user_id);
create policy "Users can block anyone by themselves. " on public.blocks for insert with check (auth.uid() = user_id);

create table if not exists public.reports (
    id uuid not null primary key DEFAULT uuid_generate_v4 (),
    user_id uuid references public.users on delete cascade not null,
    video_id uuid references public.videos on delete cascade not null,
    reason text not null,
    created_at timestamp with time zone default timezone('utc' :: text, now()) not null
);
comment on table public.reports is 'Who reported which video for what reason.';

alter table public.reports enable row level security;
create policy "Users can view their own reports." on public.reports for select using (auth.uid() = user_id);
create policy "Users can report a video." on public.reports for insert with check (auth.uid() = user_id);


create or replace view video_comments
as
    select 
        comments.id,
        comments.text,
        comments.created_at,
        comments.video_id,
        users.id as user_id,
        users.name as user_name,
        users.description as user_description,
        users.image_url as user_image_url
    from comments
    join users on comments.user_id = users.id;
    

create or replace function nearby_videos(location text, user_id uuid)
returns table(id uuid, url text, image_url text, thumbnail_url text, gif_url text, location text, created_at timestamptz, description text, user_id uuid, user_name text, user_description text, user_image_url text)
as 
$func$
    select
        videos.id,
        videos.url,
        videos.image_url,
        videos.thumbnail_url,
        videos.gif_url,
        st_astext(videos.location) as location,
        videos.created_at,
        videos.description,
        users.id as user_id,
        users.name as user_name,
        users.description as user_description,
        users.image_url as user_image_url
    from videos
    join users on videos.user_id = users.id
    where users.id not in (select blocked_user_id from blocks where user_id = user_id)
    order by location <-> st_geogfromtext($1);
$func$
language sql;

create or replace function videos_in_bouding_box(min_lng decimal, min_lat decimal, max_lng decimal, max_lat decimal, user_id uuid)
returns table(id uuid, url text, image_url text, thumbnail_url text, gif_url text, location text, created_at timestamptz, description text, user_id uuid, user_name text, user_description text, user_image_url text)
as
$func$
    select
        videos.id,
        videos.url,
        videos.image_url,
        videos.thumbnail_url,
        videos.gif_url,
        st_astext(videos.location) as location,
        videos.created_at,
        videos.description,
        users.id as user_id,
        users.name as user_name,
        users.description as user_description,
        users.image_url as user_image_url
    from videos
    join users on videos.user_id = users.id
    where users.id not in (select blocked_user_id from blocks where user_id = user_id)
    and location && ST_SetSRID(ST_MakeBox2D(ST_Point(min_lng, min_lat), ST_Point(max_lng, max_lat)),4326);
$func$
language sql;

create or replace function get_video_detail(video_id uuid, user_id uuid)
returns table(id uuid, url text, image_url text, thumbnail_url text, gif_url text, created_at timestamptz, description text, user_id uuid, user_name text, user_description text, user_image_url text, location text, like_count int, comment_count int, have_liked int)
as
$func$
    select
        videos.id,
        videos.url,
        videos.image_url,
        videos.thumbnail_url,
        videos.gif_url,
        videos.created_at,
        videos.description,
        users.id as user_id,
        users.name as user_name,
        users.description as user_description,
        users.image_url as user_image_url,
        st_astext(videos.location) as location,
        (select count(*) from likes where video_id = videos.id)::int as like_count,
        (select count(*) from comments where video_id = videos.id)::int as comment_count,
        (select count(*) from likes where video_id = videos.id and user_id = $2)::int as have_liked
    from videos
    join users on videos.user_id = users.id
    where videos.id = $1;
$func$
language sql;


create or replace view notifications
as
    select 
        'like' as type,
        videos.user_id as receiver_user_id,
        null as comment_text,
        videos.id as video_id,
        videos.thumbnail_url as video_thumbnail_url,
        likes.user_id as action_user_id,
        users.name as action_user_name,
        users.image_url as action_user_image_url,
        likes.created_at
    from likes
    join users on likes.user_id = users.id
    join videos on videos.id = likes.video_id
    union all
    select
        'comment' as type,
        videos.user_id as receiver_user_id,
        comments.text as comment_text,
        videos.id as video_id,
        videos.thumbnail_url as video_thumbnail_url,
        comments.user_id as action_user_id,
        users.name as action_user_name,
        users.image_url as action_user_image_url,
        comments.created_at
    from comments
    join users on comments.user_id = users.id
    join videos on videos.id = comments.video_id
    union all
    select
        'follow' as type,
        follow.followed_user_id as receiver_user_id,
        null as commennt_text,
        null as video_id,
        null as video_thumbnail_url,
        follow.following_user_id as action_user_id,
        users.name as action_user_name,
        users.image_url as action_user_image_url,
        follow.followed_at as created_at
    from follow
    join users on follow.following_user_id = users.id
    order by created_at desc;

-- Configure storage
insert into storage.buckets (id, name) values ('videos', 'videos');
insert into storage.buckets (id, name) values ('profiles', 'profiles');
create policy "Videos buckets are public" on storage.objects for select using (bucket_id = 'videos');
create policy "Profiles buckets are public" on storage.objects for select using (bucket_id = 'profiles');
create policy "uid has to be the first element in path_tokens" on storage.objects for insert with check (auth.uid()::text = path_tokens[1] and array_length(path_tokens, 1) = 2);


-- Needed to use extensions from the app
grant usage on schema extensions to anon;
grant usage on schema extensions to authenticated;
```

---

Android CD
https://medium.com/flutter-community/automating-publishing-your-flutter-apps-to-google-play-using-github-actions-2f67ac582032

---

[![Very Good Ventures][logo]][very_good_ventures_link]

Developed with 💙 by [Very Good Ventures][very_good_ventures_link] 🦄

<!-- ![coverage][coverage_badge] -->
[![style: very good analysis][very_good_analysis_badge]][very_good_analysis_link]
<!-- [![License: MIT][license_badge]][license_link] -->

A Very Good Flutter Starter Project created by the [Very Good Ventures Team][very_good_ventures_link].

Generated by the [Very Good CLI][very_good_cli_link] 🤖

---

## Getting Started 🚀

This project contains 3 flavors:

- development
- staging
- production

To run the desired flavor either use the launch configuration in VSCode/Android Studio or use the following commands:

```sh
# Development
$ flutter run --flavor development --target lib/main_development.dart --dart-define=SUPABASE_URL=[Supabase Url] --dart-define=SUPABASE_ANNON_KEY=[Supabase annon key]

# Staging
$ flutter run --flavor staging --target lib/main_staging.dart

# Production
$ flutter run --flavor production --target lib/main_production.dart --dart-define=SUPABASE_URL=[Supabase Url] --dart-define=SUPABASE_ANNON_KEY=[Supabase annon key]
```

_\*Spot works on iOS and Android._

---

## Running Tests 🧪

To run all unit and widget tests use the following command:

```sh
$ flutter test --coverage --test-randomize-ordering-seed random
```

To view the generated coverage report you can use [lcov](https://github.com/linux-test-project/lcov).

```sh
# Generate Coverage Report
$ genhtml coverage/lcov.info -o coverage/

# Open Coverage Report
$ open coverage/index.html
```

---

## Working with Translations 🌐

This project relies on [flutter_localizations][flutter_localizations_link] and follows the [official internationalization guide for Flutter][internationalization_link].

### Adding Strings

1. To add a new localizable string, open the `app_en.arb` file at `lib/l10n/arb/app_en.arb`.

```arb
{
    "@@locale": "en",
    "counterAppBarTitle": "Counter",
    "@counterAppBarTitle": {
        "description": "Text shown in the AppBar of the Counter Page"
    }
}
```

2. Then add a new key/value and description

```arb
{
    "@@locale": "en",
    "counterAppBarTitle": "Counter",
    "@counterAppBarTitle": {
        "description": "Text shown in the AppBar of the Counter Page"
    },
    "helloWorld": "Hello World",
    "@helloWorld": {
        "description": "Hello World Text"
    }
}
```

3. Use the new string

```dart
import 'package:spot/l10n/l10n.dart';

@override
Widget build(BuildContext context) {
  final l10n = context.l10n;
  return Text(l10n.helloWorld);
}
```

### Adding Supported Locales

Update the `CFBundleLocalizations` array in the `Info.plist` at `ios/Runner/Info.plist` to include the new locale.

```xml
    ...

    <key>CFBundleLocalizations</key>
	<array>
		<string>en</string>
		<string>es</string>
	</array>

    ...
```

### Adding Translations

1. For each supported locale, add a new ARB file in `lib/l10n/arb`.

```
├── l10n
│   ├── arb
│   │   ├── app_en.arb
│   │   └── app_es.arb
```

2. Add the translated strings to each `.arb` file:

`app_en.arb`

```arb
{
    "@@locale": "en",
    "counterAppBarTitle": "Counter",
    "@counterAppBarTitle": {
        "description": "Text shown in the AppBar of the Counter Page"
    }
}
```

`app_es.arb`

```arb
{
    "@@locale": "es",
    "counterAppBarTitle": "Contador",
    "@counterAppBarTitle": {
        "description": "Texto mostrado en la AppBar de la página del contador"
    }
}
```

[coverage_badge]: coverage_badge.svg
[flutter_localizations_link]: https://api.flutter.dev/flutter/flutter_localizations/flutter_localizations-library.html
[internationalization_link]: https://flutter.dev/docs/development/accessibility-and-localization/internationalization
[license_badge]: https://img.shields.io/badge/license-MIT-blue.svg
[license_link]: https://opensource.org/licenses/MIT
[logo]: https://raw.githubusercontent.com/VeryGoodOpenSource/very_good_analysis/main/assets/vgv_logo.png
[very_good_analysis_badge]: https://img.shields.io/badge/style-very_good_analysis-B22C89.svg
[very_good_analysis_link]: https://pub.dev/packages/very_good_analysis
[very_good_cli_link]: https://github.com/VeryGoodOpenSource/very_good_cli
[very_good_ventures_link]: https://verygood.ventures/?utm_source=github&utm_medium=banner&utm_campaign=core
