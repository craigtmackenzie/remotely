# Remotely

[![Build Status](https://secure.travis-ci.org/wegowise/remotely.png)](http://travis-ci.org/wegowise/remotely)

Remotely lets you specify associations for your models that should
be fetched from a remote API instead of the database.

## Configuration

Apps are where Remotely goes to find association resources. You can define as many as you want, but if you define only one, you can omit the `:app` option from your associations.

    Remotely.configure do
      app :legsapp do
        url "http://somanylegs.com/api/v1"
        basic_auth "username", "password"
      end
    end

## Defining Associations

`has_many_remote` takes two options, `:app` and `:path`. `:app` tells Remotely which registered app to fetch it from. `:path` tells it the URI to the object (everything after the app).

**One app & association name matches URI**

    class Millepied < ActiveRecord::Base
      has_many_remote :legs # => "/legs"
    end

**One app & custom path**

    class Millepied < ActiveRecord::Base
      has_many_remote :legs, :path => "/store/legs"
    end

**One app & custom path with `id` substitution**

    class Millepied < ActiveRecord::Base
      has_many_remote :legs, :path => "/millepieds/:id/legs"
    end

**Multiple apps (all secondary conditions from above apply)**

    class Millepied < ActiveRecord::Base
      has_many_remote :legs, :app => :legsapp, ...
    end

**Note about associations**

If you do not specify the `:app` options in your associations, you need
to create a `Remotely::Model` subclass for the associated object. This
is so Remotely knows which app to use to retrieve entries of that type.

    class Person < ActiveRecord::Base
      has_many_remote :legs
    end

    # Means the following must exist:

    class Leg < Remotely::Model
      app :legsapp
      uri "/legs"
    end

### id Substitution

A path can include "`:id`" anywhere in it, which is replaced by the instance's `id`. This is useful when the resource on the API end is namespaced. For example:

    class Millepied < ActiveRecord::Base
      has_many_remote :legs, :path => "/millepieds/:id/legs"
    end

    m = Millepied.new
    m.id   # => 1
    m.legs # => Requests "/millepieds/1/legs"

## Fetched Objects

Remote associations are Remotely::Model objects. Whatever data the API returns, becomes the attributes of the Model.

    m = Millepied.new
    m.legs[0]         # => #<Remotely::Model:0x0000f351c8 @attributes={:length=>"1mm"}>
    m.legs[0].length  # => "1mm"

### Fetched Object Associations

If a fetched object includes an attribute matching "\*_id", Remotely tries to find the model it is for and retrieve it.

    leg = m.legs.first
    leg.user_id # => 2
    leg.user    # => User.find(2)

## Contributing

Fork, branch and pull-request. Bump versions in their own commit.
