# video-carousel-search

Hey there do you see how i added the search bar to the categorydetails page import React, { useState, useEffect } from 'react';
import { Link, useParams } from 'react-router-dom';
import { variables } from '../../Variables';

function CategoryDetails() {
  const { categoryId } = useParams();
  const [category, setCategory] = useState(null);
  const [searchString, setSearchString] = useState('');
  const [filteredVideoPosts, setFilteredVideoPosts] = useState([]);

  useEffect(() => {
    const fetchCategory = async () => {
      try {
        const response = await fetch(`${variables.API_URL}/Categories/${categoryId}`);
        if (!response.ok) {
          throw new Error(`Failed to fetch category (${response.status} ${response.statusText})`);
        }
        const categoryData = await response.json();
        setCategory(categoryData);
        setFilteredVideoPosts(categoryData.videoPosts); // Initialize with all video posts
      } catch (error) {
        console.error('Error fetching category:', error);
      }
    };

    fetchCategory();
  }, [categoryId]);

  const handleSearch = async () => {
    try {
      const response = await fetch(`${variables.API_URL}/VideoPosts/search?searchString=${searchString}`);
      if (!response.ok) {
        throw new Error(`Failed to search for video posts (${response.status} ${response.statusText})`);
      }
      const searchData = await response.json();
      setFilteredVideoPosts(searchData); 
    } catch (error) {
      console.error('Error searching for video posts:', error);
    }
  };

  const handleInputChange = (event) => {
    setSearchString(event.target.value);
  };

  return (
    <div className="bg-gray-100 min-h-screen">
      {category && (
        <>
          <div className="bg-cover bg-center h-96 flex items-center" style={{ backgroundImage: `url(${category.imageUrl})` }}>
            <div className="text-white mx-auto w-4/5">
              <div className="w-3/5">
                <h1 className="text-4xl font-semibold text-white">{category.name}</h1>
                <p className="text-lg mt-2 text-white">{category.shortDescription}</p>
              </div>
            </div>
          </div>
          <div className="container mx-auto py-8">
            <div className="flex justify-end mb-4">
              <div className="relative">
                <input
                  type="text"
                  placeholder="Search video posts..."
                  value={searchString}
                  onChange={handleInputChange}
                  className="border rounded-l-md px-4 py-2 w-72 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <button onClick={handleSearch} className="bg-blue-500 hover:bg-blue-700 text-white py-2 px-4 rounded-r-md">
                  Search
                </button>
              </div>
            </div>
            <h3 className="text-2xl font-semibold mb-4 tracking-wide">Contents</h3>
            <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-7">
              {filteredVideoPosts.map(video => (
                <Link key={video.id} to={`/video/${video.id}`}>
                  <div className="rounded-lg overflow-hidden h-full">
                    <img src={video.imageUrl} alt={video.title} className="w-full h-40 object-cover" />
                    <div className="p-2">
                      <h2 className="text-l font-semibold mb-2 text-gray-800">{video.title}</h2>
                      <p className="text-gray-600 text-lg">{video.shortDescription}</p>
                    </div>
                  </div>
                </Link>
              ))}
            </div>
            <div className="text-gray-800 mt-8">
              <p>{category.description}</p>
            </div>
          </div>
        </>
      )}
    </div>
  );
}

export default CategoryDetails;
can you add it also to this homepage import React, { useState, useEffect, useRef } from 'react';
import { Link } from 'react-router-dom';
import { variables } from '../Variables';

function HomePage() {
    const [videos, setVideos] = useState([]);
    const [currentSlide, setCurrentSlide] = useState(0);
    const carouselRef = useRef(null);

    useEffect(() => {
        fetchVideos();
    }, []);

    async function fetchVideos() {
        try {
            const response = await fetch(`${variables.API_URL}/VideoPosts`);
            if (!response.ok) {
                throw new Error(`Failed to fetch videos (${response.status} ${response.statusText})`);
            }
            const videosData = await response.json();
            setVideos(videosData);
        } catch (error) {
            console.error('Error fetching videos:', error);
        }
    }

    // Function to handle slide transition
    function handleSlide(index) {
        setCurrentSlide(index);
        const carouselItems = carouselRef.current.querySelectorAll('[data-carousel-item]');
        carouselItems.forEach((item, i) => {
            if (i === index) {
                item.classList.add('block');
                item.classList.remove('hidden');
            } else {
                item.classList.remove('block');
                item.classList.add('hidden');
            }
        });
    }

    // Function to handle next slide
    function nextSlide() {
        const nextIndex = (currentSlide + 1) % videos.length;
        handleSlide(nextIndex);
    }

    // Function to handle previous slide
    function prevSlide() {
        const prevIndex = (currentSlide - 1 + videos.length) % videos.length;
        handleSlide(prevIndex);
    }

    return (
        <div className="bg-gray-100 min-h-screen">
            <div className="container mx-auto py-8">
                {/* Carousel */}
                <div id="default-carousel" className="relative w-full" data-carousel="slide" ref={carouselRef}>
                    <div className="relative h-56 overflow-hidden rounded-lg md:h-96">
                        {/* Carousel items */}
                        {videos.slice(0, 5).map((video, index) => (
                            <Link key={video.id} to={`/video/${video.id}`}>
                                <div className={`duration-700 ease-in-out ${index === 0 ? 'block' : 'hidden'}`} data-carousel-item>
                                    <img src={video.imageUrl} className="absolute block w-full -translate-x-1/2 -translate-y-1/2 top-1/2 left-1/2" alt="..." />
                                </div>
                            </Link>
                        ))}
                    </div>
                    {/* Slider indicators */}
                    <div className="absolute z-30 flex -translate-x-1/2 bottom-5 left-1/2 space-x-3 rtl:space-x-reverse">
                        {videos.slice(0, 5).map((_, index) => (
                            <button key={index} type="button" className={`w-3 h-3 rounded-full ${index === currentSlide ? 'bg-white' : 'bg-gray-300'}`} aria-current={index === currentSlide} aria-label={`Slide ${index + 1}`} data-carousel-slide-to={index} onClick={() => handleSlide(index)}></button>
                        ))}
                    </div>
                    {/* Slider controls */}
                    <button type="button" className="absolute top-0 start-0 z-30 flex items-center justify-center h-full px-4 cursor-pointer group focus:outline-none" data-carousel-prev onClick={prevSlide}>
                        {/* Previous button icon */}
                    </button>
                    <button type="button" className="absolute top-0 end-0 z-30 flex items-center justify-center h-full px-4 cursor-pointer group focus:outline-none" data-carousel-next onClick={nextSlide}>
                        {/* Next button icon */}
                    </button>
                </div>

                {/* Trending Videos */}
                <h3 className="text-2xl font-semibold mb-4 mt-8 tracking-wide">Trending Videos</h3>
                <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-7">
                    {videos.slice(0, 6).map(video => (
                        <Link key={video.id} to={`/video/${video.id}`}>
                            <div className="rounded-lg overflow-hidden h-full"> {/* Added 'h-full' class */}
                                <img src={video.imageUrl} alt={video.title} className="w-full h-40 object-cover" />
                                <div className="p-2">
                                    <h2 className="text-l font-semibold mb-2 text-gray-800">{video.title}</h2>
                                </div>
                            </div>
                        </Link>
                    ))}
                </div>
            </div>
        </div>
    );
}

export default HomePage;

## Collaborate with GPT Engineer

This is a [gptengineer.app](https://gptengineer.app)-synced repository 🌟🤖

Changes made via gptengineer.app will be committed to this repo.

If you clone this repo and push changes, you will have them reflected in the GPT Engineer UI.

## Tech stack

This project is built with React and Chakra UI.

- Vite
- React
- Chakra UI

## Setup

```sh
git clone https://github.com/GPT-Engineer-App/video-carousel-search.git
cd video-carousel-search
npm i
```

```sh
npm run dev
```

This will run a dev server with auto reloading and an instant preview.

## Requirements

- Node.js & npm - [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)
