## Hi there 👋 I am Jai Gora

#### Student at Shri Mata Vaishno Devi University
 * A curious programmer💻🎧
 * Software Developer </> 💻
 * Open source contributor 💻
 * Creative mind 💭

📫 Social profiles:

<a href="https://www.linkedin.com/in/jai-gora-893343194/">
  <img src="https://github.com/dmhendricks/signature-social-icons/blob/master/icons/round-flat-filled/35px/linkedin.png" alt="LinkedIn" title="LinkedIn" width="35" height="35" />
</a>
<a href="https://twitter.com/jaigora24/">
  <img src="https://cdn.jsdelivr.net/gh/dmhendricks/signature-social-icons/icons/round-flat-filled/50px/twitter.png" alt="Twitter" title="Twitter" width="35" height="35" />
</a>
<a href="https://www.instagram.com/jaigora24/">
  <img src=http://assets.stickpng.com/thumbs/580b57fcd9996e24bc43c521.png alt="Instagram" title="Instagram" width="35" height="35" />
  </a>
<a href="https://jaigora24.blogspot.com/">
  <img src="https://www.lifewire.com/thmb/v4P_CNrqWGsQrdj6RaANe2XSVbk=/768x0/filters:no_upscale():max_bytes(150000):strip_icc()/Blogger.svg-57f268d63df78c690fe5d003.png" alt="Twitter" title="Blogger" width="35" height="35" />
  </a>


const { request } = require("./utils");
const retryer = require("./retryer");
require("dotenv").config();

const fetcher = (variables, token) => {
  return request(
    {
      query: `
      query userInfo($login: String!) {
        user(login: $login) {
          repositories(isFork: false, first: 100) {
            nodes {
              languages(first: 1) {
                edges {
                  size
                  node {
                    color
                    name
                  }
                }
              }
            }
          }
        }
      }
      `,
      variables,
    },
    {
      Authorization: `bearer ${token}`,
    }
  );
};

async function fetchTopLanguages(username) {
  if (!username) throw Error("Invalid username");

  let res = await retryer(fetcher, { login: username });

  if (res.data.errors) {
    console.log(res.data.errors);
    throw Error(res.data.errors[0].message || "Could not fetch user");
  }

  let repoNodes = res.data.data.user.repositories.nodes;

  // TODO: perf improvement
  repoNodes = repoNodes
    .filter((node) => {
      return node.languages.edges.length > 0;
    })
    .sort((a, b) => {
      return b.languages.edges[0].size - a.languages.edges[0].size;
    })
    .map((node) => {
      return node.languages.edges[0];
    })
    .reduce((acc, prev) => {
      let langSize = prev.size;
      if (acc[prev.node.name] && prev.node.name === acc[prev.node.name].name) {
        langSize = prev.size + acc[prev.node.name].size;
      }

      return {
        ...acc,
        [prev.node.name]: {
          name: prev.node.name,
          color: prev.node.color,
          size: langSize,
        },
      };
    }, {});

  const topLangs = Object.keys(repoNodes)
    .slice(0, 5)
    .reduce((result, key) => {
      result[key] = repoNodes[key];
      return result;
    }, {});

  return topLangs;
}

module.exports = fetchTopLanguages;
