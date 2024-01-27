# MongoDB aggregation

## How many users are active?

```bash
[
	// first pipeline
  {
    $match: {
      isActive: true
    }
  },
	// second pipeline
  {
		$count: "activeUsers"
  }
]
```

- `$match` and `$count` are MongoDB operators.
- The above code is the first pipeline
- second pipeline will work on the filtered-out data from the first pipeline **not** on the whole data.

## What is the average age of all users?

```bash
[
  {
    $group: {
      _id: null,
      averageAge:{
        $avg: "$age"
      }
    }
  }
]
```

- here we are using group operator to group fields.
- \_id: null means we are not grouping based on a particular field rather combining all irrespective of the field. In short all the documents will be combined within 1 document.
- In the another field we can pass a accumulator.
- In the next we are creating a new field averageAge and in that I have added `$avg` accumulator
- `$avg` does avg based on age over here.

## List the top 2 most common fruit

```bash
[
  {
    $group: {
      _id: "$favoriteFruit",
      count:{
        $sum: 1
      }
    }
  },
  {
    $sort: {
      count: -1
    }
  },
  {
    $limit: 2
  }
]
```

- In the `$group` pipeline, we are finding the count of different fruits.
- $sum: 1 means increment by one to the previous value and sum starts from 0.
- In the `$sort` pipeline I am sorting the above data in descending order as count: -1 and for ascending order put 1.
- Finally, in the **`$limit`** pipeline, we are limiting the output to a specific value.

# Find total number of males and females

```bash
[
  {
    $group: {
// grouped based on gender
      _id: "$gender",
// counted the number of documents in each group
      count: {
        $sum: 1,
      },
    },
  },
]
```

# Which county has highest number of registered user?

```bash
[
  {
    $group: {
      _id: "$company.location.country",
      count: {
        $sum: 1,
      },
    },
  },
  {
    $sort: {
      count: -1,
    },
  },
  {
    $limit: 1,
  },
]
```

# List unique eye color in a collection

```bash
[
  {
    $group: {
      _id: "$eyeColor",
    },
  },
]
```

# Average number of tags per user

## Method-1

```bash
[
  {
    $unwind: "$tags",
  },
  {
    $group: {
      _id: "$_id",
      count: {
        $sum: 1,
      },
    },
  },
  {
    $group: {
      _id: null,
      avg: {
        $avg: "$count",
      },
    },
  },
]
```

- `unwind` splits the tags array, and it creates copies of the document which is equal to the number of tags and in each document a single tag will be present for ex: if there are 5 tags then it will create 5 documents in which one document will contain one tag and rest part of the document remains same.
- Then in the next pipleline we are grouping the documents on the basis of there \_id and counting them.
- finally we get all the tags with their count and then we will find average by grouping all documents into one and using the `avg` mongodb operator

## Method-2

```bash
[
  {
    $addFields: {
      numberOfTags: {
        $size: {
          $ifNull: ["$tags", []],
        },
      },
    },
  },
  {
    $group: {
      _id: null,
      averageNumberOfTags: {
        $avg: "$numberOfTags",
      },
    },
  },
]
```

- `addFields`: is used to add a new field to all the documents, here `numberOfTags` is created where `$size` is used to find the size of the array and we are using `$ifNull` to check whether there is null value in place of array and if null is there then treat it as empty array.
- then finding the average of the `numberOfTags` field by clubbing all the documents into one and finding the average

# find the number of users with enim as tag

```bash
[
  {
    $match: {
      tags:"enim"
    }
  },
  {
    $count: 'userWithEnim'
  }
]
```

- `$match` operator filters out the users with **enim** as one of the tags
- In the next pipeline we are taking the filtered out users as input and then we are counting the documents.

# What are the names and age of users who are inactive and have 'velit' as a tag?

```bash
[
  {
    $match: {
      tags:"velit",
      isActive: false,
    }
  },
  {
		$project: {
		  name:1,
      age:1
		}
  }
]
```

- `$match` operator filters out the users with **velit** as one of the tags.
- In the next stage or pipeline we are using the filtered out data
- `$project` operator is used to extract a certain existing field or to add a new field by computing it.
- here we are extracting **name** and **age** field from the whole document.

# How many users have a phone number starting with '+1 (940)'?

```bash
[
  {
    $match: {
      "company.phone": /^\+1 \(940\)/
    }
  },
  {
    $count: 'userCount'
  }
]
```

- In the first pipeline I have used `$match` with regex to filter out the users with the phone number starting with **+1 (940)**.
- In the second pipeline we have to count them so used `$count` for that.

# Who has registered the most recently?

```bash
[
  {
    $sort: {
      registered: -1
    }
  },
  {
    $limit: 4
  },
  {
    $project: {
      name:1,
      registered:1,
      favoriteFruit:1
    }
  }
]
```

- **First pipeline**: sort the document bases on the registered field with -1 indicating the recent registered to appear at the top.
- **Second pipeline**: limit the sorted documents to a random number here 4.
- **Third pipeline**: Projected few fields from the above pipeline.

# Categorize users by their favorite fruit.

```bash
[
  {
    $group: {
      _id: "$favoriteFruit",
      users:{
        $push:"$name"
      }
    }
  }
]
```

- Grouping the users based on the **favoriteFruit** field.
- Then added another field users and pushed all the users of a particular **favoriteFruit** in that using `$push` operator which pushes a array.

# How many users have 'ad' as the second tag in their list of tags?

```bash
[
  {
    $match: {
      "tags.1": "ad",
    },
  },
  {
		$count: 'usersCount'
  }
]
```

- First pipeline we are filtering out the documents based on ad as the second element of the tag array.
- In the second pipeline we are counting the number of documents

# Find users wmo have both 'enim' and 'id' as their tags.

```bash
[
  {
    $match: {
      tags:{
        $all:["id","enim"]
      }
    }
  },
]
```

- Filtering the users based on tags
- We need to check that users should have **enim** as well as **id** in their tags
- `$all` operator checks that all the provided fields are present.

# List all companies located in the USA with their corresponding user count.

```bash
[
  {
    $match: {
      "company.location.country": "USA"
    }
  },
  {
    $group: {
      _id: "$company.title",
      employeeCount:{
        $sum:1
      }
    }
  }
]
```

1. **$match Stage:**
   - This stage filters documents based on a specified condition.
   - In this case, it filters documents where the value of the "company.location.country" field is "USA".
2. **$group Stage:**
   - This stage groups the documents by the specified field, which is "\_id: "$company.title"" in this case.
   - For each group, it calculates the total count of documents using the $sum operator, and the result is assigned to the field "employeeCount".

So, the overall purpose of this pipeline is to find and aggregate the number of employees for each unique company title where the company is located in the USA. The result will include the unique company titles and the corresponding total employee count for each title.
