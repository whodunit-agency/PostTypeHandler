# PostTypeHandler
Helper class to quickly manage PostType and Taxonomy declarations

## Features
- Easily add new Post Types
- Easily add new Taxonomies
- Easily link Post Types to Taxonomies & vice versa
- Easily add new columns to the admin and manage them ( populate, sort )

## Installation

**Install with composer**

Run the following in your terminal to install the package with composer

```sh
composer require gturpin/post-type-handler
```

The package use the autoloader, so don't forget to register the autoloader.  
If you don't know how see the basic example below.

## Basic Usage
Below is a basic example of setting up a simple PostType.

```php
// don't forget to import your autoloader
require_once __DIR__ . '/vendor/autoload.php';

use PostTypeHandler\PostType;

$post_type_handler = new PostType( 'Book' );
$post_type_handler->register();
```

You can add custom $options and $labels to the PostType declaration.

```php
$labels = [
	'add_new'   => __( 'my add_new', 'context' ),
	'all_items' => __( 'my all_items', 'context' ),
];

$options = [
	'public'    => true,
	'menu_icon' => 'dashicons-admin-post',
	'rewrite'   => [
		'slug' => 'my-post-type',
	],
];

$post_type_handler = new PostType( 'Books', $options, $labels );
$post_type_handler->register();
```

You can also set the taxonomies for the PostType if they are previously registered.

```php
$post_type_handler = new PostType( 'Books' );

// add multiple taxonomies
$post_type_handler->set_taxonomies( [ 'custom-taxonomy', 'post_tag' ] );

// or add a single taxonomy
$post_type_handler->set_taxonomies( 'custom-taxonomy' );

$post_type_handler->register();
```

Otherwise you can register a new Taxonomy and then even add it to the PostType declaration.

```php
// Register the Taxonomy
$taxonomy_handler = new Taxonomy( 'custom-taxonomy' );
$taxonomy_handler->register();

// Add it to the PostType in PostType declaration
$post_type_handler = new PostType( 'Books' );
$post_type_handler->set_taxonomies( 'custom-taxonomy' );
$post_type_handler->register();
```

Or you can set the taxonomy to a Post Type that is already registered.

```php
$taxonomy_handler = new Taxonomy( 'custom-taxo' );
$taxonomy_handler->set_post_types( 'books' ); // work aswell with an array : [ 'books', $post_type_object ]
$taxonomy_handler->register();
```

You can give the Taxonomy object itself to the PostType.

```php
$taxonomy_handler = new Taxonomy( 'custom-taxo' );
$taxonomy_handler->register();

$post_type_handler = new PostType( 'Books' );
$post_type_handler->set_taxonomies( $taxonomy_handler ); // work aswell with an array : [ 'post_tag', $taxonomy_handler ]
$post_type_handler->register();
```

## Manage Post Types columns
I will explain some examples of how to manage the columns for a Post Type.  

Firstly, you need to register the Post Type.  
Then you can do stuff with the columns.  
Finally you need to save the changes by registering the Post Type.

### Add a new column
To add new columns to a Post Type you can do the following

```php
$post_type_handler = new PostType( 'Books' );

// Call the columns function to get access to the column manager and add a new column
$post_type_handler->columns()->add( [
	'custom-slug' => __( 'Custom label', 'context' ),
	'year'        => __( 'Year', 'context' ),
] );

// You can also pass only one slug and label
$post_type_handler->columns()->add( 'custom-slug', 'Custom label' );
$post_type_handler->register();
```

### Hide a column
To hide a column you can do the following

```php
$post_type_handler = new PostType( 'Books' );

// Call the columns function to get access to the column manager and hide a built-in column or a custom one
$post_type_handler->columns()->hide( [
	'custom-slug',
	'date'
] );

// You can also hide only one column
$post_type_handler->columns()->hide( 'year' );
$post_type_handler->register();
```

### Set all columns
You can set all columns at once  
By doing this you must take a look at the [Manage columns hook](https://developer.wordpress.org/reference/hooks/manage_post_type_posts_columns/) to prevent unwanted columns

```php
$post_type_handler = new PostType( 'Books' );

// Call the columns function to get access to the column manager and set all columns
$post_type_handler->columns()->set( [
	'custom-slug' => 'Custom label',
	'year'        => 'Year',
] );

$post_type_handler->register();
```

### Populate a column
To populate a column you can do the following  
Note that you can only populate one column at once  
Note that you must display the content and not return it  
> You can't use this to populate a [built-in column](https://developer.wordpress.org/reference/hooks/manage_post_type_posts_columns/#more-information)

```php
$post_type_handler = new PostType( 'Books' );

// Call the columns function to get access to the column manager and populate a column
$post_type_handler->columns()->populate( 'custom-slug', function( $column, $post_id ) {
	echo get_the_title( $post_id );
} );

$post_type_handler->register();
```

### Make a column sortable
To make a column sortable you can do the following  
Note that you must make the column slug in key and value of the array  
The value must be the name of the meta key  
Don't forget to populate the column before you make it sortable  

```php
$post_type_handler = new PostType( 'Books' );

// Call the columns function to get access to the column manager and make a column sortable
$post_type_handler->columns()->sortable( [
	'rating' => 'rating',
	'year'   => [ 'year', true ], // You can add true to make the sort by numeric order or false to make it by alphabetical order which is the default
] );

$post_type_handler->register();
```

## Hooks

| Hook type | Hook name                         | Params         | Description                          |
| --------- | --------------------------------- | -------------- | ------------------------------------ |
| Filter    | gt_post_type_{$post_type}_labels  | array $labels  | Custom the labels for the post type  |
| Filter    | gt_post_type_{$post_type}_options | array $options | Custom the options for the post type |
| Filter    | gt_taxonomy_{$post_type}_labels   | array $labels  | Custom the labels for the taxonomy   |
| Filter    | gt_taxonomy_{$post_type}_options  | array $options | Custom the options for the taxonomy  |


## TODOS

- ~~Can also add taxonomy by sending the object itself ( by the object itself, maybe with a __tostring method )~~
- Adding a way to manage Columns
  - ~~Hide columns and defaults for each post type~~
  - ~~Adding new columns to the admin screen~~
  - Set columns order
  - ~~Set the entire columns array~~
  - ~~Populate any column with a custom function~~
  - ~~Can sort each columns with their values ( numerically / alphabetically )~~
- Adding a function to easily add icon without using the $options array
- Adding a way to manage the Filters on screen admin
  - Set an array to order them and keep an order
- ~~Add a class to manage the taxonomies~~
  - ~~Adding new Taxonomies~~
  - Can work on existing taxonomies ( post_tag & category )
  - ~~Can be registered on a post type directly ( by the slug or the object itself, maybe with a __tostring method )~~
- Same columns but for the taxonomies
- Can work on existing post types ( update options and labels )
- Add the @link/author/license to the main class