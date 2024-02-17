# MONGODB AGGREGATION PIPELINE

## LOOKUP
## Lookup performs left outer join so this means that all documents from the source collection will be included in the result, and if there are no matching documents in the destination collection, the result will still include the source document with an empty or null value for the joined field.

```javascript
const user = await User.aggregate([
    {
      //get watch history of the currently logged in user so match for against
      //his id.
      $match: {
        _id: new mongoose.Types.ObjectId(req.user._id),
      },
    },
    {
      //get videos details that are watched by the current/logged in user
      $lookup: {
        from: "videos",
        localField: "watchHistory", //already gets populated.
        foreignField: "_id",
        as: "watchHistory",
        pipeline: [
          {
            //to get owner of the videos that are matched above.
            $lookup: {
              from: "users",
              localField: "owner", // (also a user) under videos => owner field
              foreignField: "_id", //(user) under user => _id field 
              as: "owner",
              //get the required output
              pipeline: [
                {
                  $project: {
                    fullName: 1,
                    username: 1,
                    avatar: 1,
                  },
                },
              ],
            },
          },
          /*just to get the first object from 
          the output array.*/
          {
            $addFields:{
              owner:{
                $first : "$owner"
              }
            }
          }
        ],
      },
    },
  ]);
```

### MongoDB Aggregation Pipeline Summary

1. **$match Stage:**
   - Filters the "User" documents based on the logged-in user's `_id`.
   - **Outcome:** Single user document matching the logged-in user.

2. **$lookup Stage (first level):**
   - Performs a left outer join with the "videos" collection using the "watchHistory" field in the "User" collection.
   - Retrieves details about the videos watched by the logged-in user.
   - **Outcome:** User document with an enriched "watchHistory" array containing details about watched videos.

3. **$lookup Inside Nested Pipeline:**
   - Within the "watchHistory" array, performs another left outer join with the "users" collection.
   - Joins based on the "owner" field in the "videos" collection and the "_id" field in the "users" collection.
   - Retrieves details about the users who uploaded each video.
   - **Outcome:** User document with "watchHistory" array, where each video includes details about its owner.

4. **$project Inside Nested Pipeline:**
   - Projects specific fields from the "owner" field, such as "fullName," "username," and "avatar."
   - **Outcome:** User document with further refined "watchHistory" array, containing videos with owner details.

5. **$addFields Inside Nested Pipeline:**
   - Adds a new field called "owner" to each document in the "watchHistory" array.
   - Extracts the first element from the "owner" array to simplify the structure.
   - **Outcome:** Final user document with a fully enriched "watchHistory" array, where each video includes details about its owner.

The final result is a user document containing information about the videos watched by the logged-in user, and each video is enriched with details about its owner, providing a comprehensive set of data for the user's watch history.
