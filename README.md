Project Structure
text
idpedia-backend/
├── src/
│   ├── controllers/
│   │   └── ideaController.ts
│   ├── services/
│   │   ├── conceptService.ts
│   │   └── gateEngineService.ts
│   ├── models/
│   │   └── conceptModel.ts
│   ├── routes/
│   │   └── apiRoutes.ts
│   ├── utils/
│   │   └── noveltyScorer.ts
│   └── app.ts
├── package.json
├── tsconfig.json
└── README.md

1. Package.json
json
{
  "name": "idpedia-backend",
  "version": "1.0.0",
  "main": "dist/app.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/app.js",
    "dev": "ts-node-dev src/app.ts"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/cors": "^2.8.15",
    "typescript": "^5.2.2",
    "ts-node-dev": "^2.0.0"
  }
}

2. Concept Model (src/models/conceptModel.ts)
export interface Concept {
  id: string;
  name: string;
  domain: string; // e.g., "materials", "electronics", "biology", "energy"
  tags: string[];
  description: string;
}

export const conceptDatabase: Concept[] = [
  {
    id: "c1",
    name: "biodegradable materials",
    domain: "materials",
    tags: ["eco-friendly", "decomposable", "sustainable"],
    description: "Materials that break down naturally in the environment."
  },
  {
    id: "c2",
    name: "wireless charging",
    domain: "electronics",
    tags: ["energy transfer", "induction", "cordless"],
    description: "Transmitting power without physical connectors."
  },
  {
    id: "c3",
    name: "neural networks",
    domain: "AI",
    tags: ["machine learning", "pattern recognition", "adaptive"],
    description: "Computational systems inspired by biological brains."
  },
  {
    id: "c4",
    name: "self-healing polymers",
    domain: "materials",
    tags: ["repair", "durability", "smart materials"],
    description: "Materials that autonomously repair damage."
  },
  {
    id: "c5",
    name: "kinetic energy harvesting",
    domain: "energy",
    tags: ["movement", "wearable", "self-powered"],
    description: "Capturing energy from motion."
  },
  {
    id: "c6",
    name: "blockchain",
    domain: "technology",
    tags: ["decentralized", "ledger", "trustless"],
    description: "Distributed tamper-proof record keeping."
  }
];

3. Novelty Scorer (src/utils/noveltyScorer.ts)
import { Concept } from "../models/conceptModel";

export interface ScoredIdea {
  id: string;
  name: string;
  description: string;
  noveltyScore: number; // 0-100
  combinedConcepts: string[];
  gateUsed: "AND" | "OR" | "NOT";
}

// Heuristic: less common tag overlap → higher novelty
export function calculateNovelty(
  concepts: Concept[],
  ideaName: string
): number {
  const allTags = concepts.flatMap(c => c.tags);
  const uniqueTags = new Set(allTags);
  
  // Novelty increases with tag diversity
  const tagDiversity = uniqueTags.size / (allTags.length + 1);
  
  // Penalize very obvious naming patterns
  const obviousWords = ["smart", "system", "device", "new"];
  const obviousPenalty = obviousWords.some(word => ideaName.toLowerCase().includes(word)) ? 15 : 0;
  
  let score = 40 + (tagDiversity * 60) - obviousPenalty;
  return Math.min(100, Math.max(0, Math.floor(score)));
}

4. Gate Engine Service (src/services/gateEngineService.ts)
import { Concept } from "../models/conceptModel";
import { calculateNovelty, ScoredIdea } from "../utils/noveltyScorer";

export class GateEngine {
  
  // AND gate: All concepts must be present. Idea = intersection + emergent property
  static andGate(concepts: Concept[]): ScoredIdea {
    const combinedName = concepts.map(c => c.name).join(" + ");
    const ideaId = `and_${Date.now()}_${Math.random().toString(36).substr(2, 6)}`;
    
    // Synthesize description by merging domain features
    const description = `System integrating ${concepts.map(c => c.name).join(" and ")}. ` +
      `Emergent property: ${concepts[0].domain}-driven ${concepts[1]?.domain || "adaptive"} solution.`;
    
    const noveltyScore = calculateNovelty(concepts, combinedName);
    
    return {
      id: ideaId,
      name: `${combinedName} hybrid`,
      description,
      noveltyScore,
      combinedConcepts: concepts.map(c => c.name),
      gateUsed: "AND"
    };
  }
  
  // OR gate: Idea can be satisfied by any concept → often broader applications
  static orGate(concepts: Concept[]): ScoredIdea {
    const ideaId = `or_${Date.now()}_${Math.random().toString(36).substr(2, 6)}`;
    const name = `Versatile ${concepts.map(c => c.name).join(" or ")} platform`;
    
    const description = `A modular system capable of functioning as ${concepts.map(c => c.name).join(" OR ")}. ` +
      `Switchable behavior based on context.`;
    
    const noveltyScore = calculateNovelty(concepts, name);
    
    return {
      id: ideaId,
      name,
      description,
      noveltyScore,
      combinedConcepts: concepts.map(c => c.name),
      gateUsed: "OR"
    };
  }
  
  // NOT gate: Idea excludes a specific concept (uses A but NOT B)
  static notGate(includeConcept: Concept, excludeConcept: Concept): ScoredIdea {
    const ideaId = `not_${Date.now()}_${Math.random().toString(36).substr(2, 6)}`;
    const name = `${includeConcept.name} without ${excludeConcept.name}`;
    
    const description = `Innovative approach leveraging ${includeConcept.name} but explicitly avoiding ` +
      `${excludeConcept.name}. Solves limitations of traditional ${excludeConcept.name}-based designs.`;
    
    const noveltyScore = calculateNovelty([includeConcept, excludeConcept], name);
    
    return {
      id: ideaId,
      name,
      description,
      noveltyScore,
      combinedConcepts: [includeConcept.name, `NOT ${excludeConcept.name}`],
      gateUsed: "NOT"
    };
  }
  
  // Multi-concept combiner (arbitrary AND/OR mix via templates)
  static customCombination(concepts: Concept[], mode: "strict_and" | "any_or"): ScoredIdea {
    if (mode === "strict_and") {
      return this.andGate(concepts);
    } else {
      return this.orGate(concepts);
    }
  }
}

5. Concept Service (src/services/conceptService.ts)
import { Concept, conceptDatabase } from "../models/conceptModel";

export class ConceptService {
  static getAllConcepts(): Concept[] {
    return conceptDatabase;
  }
  
  static getConceptById(id: string): Concept | undefined {
    return conceptDatabase.find(c => c.id === id);
  }
  
  static getConceptsByDomain(domain: string): Concept[] {
    return conceptDatabase.filter(c => c.domain.toLowerCase() === domain.toLowerCase());
  }
  
  static searchConcepts(query: string): Concept[] {
    const lowerQuery = query.toLowerCase();
    return conceptDatabase.filter(c => 
      c.name.includes(lowerQuery) || 
      c.tags.some(tag => tag.includes(lowerQuery)) ||
      c.domain.includes(lowerQuery)
    );
  }
  
  static addConcept(concept: Omit<Concept, "id">): Concept {
    const newConcept: Concept = {
      ...concept,
      id: `c${conceptDatabase.length + 1}`
    };
    conceptDatabase.push(newConcept);
    return newConcept;
  }
}

6. Idea Controller (src/controllers/ideaController.ts)
import { Request, Response } from "express";
import { ConceptService } from "../services/conceptService";
import { GateEngine } from "../services/gateEngineService";
import { ScoredIdea } from "../utils/noveltyScorer";

export class IdeaController {
  
  // Generate ideas from 2+ concepts using AND gate
  static async combineAND(req: Request, res: Response) {
    try {
      const { conceptIds } = req.body; // array of concept IDs
      if (!conceptIds || conceptIds.length < 2) {
        return res.status(400).json({ error: "Need at least 2 concepts for AND gate" });
      }
      
      const concepts = conceptIds.map((id: string) => ConceptService.getConceptById(id)).filter(c => c);
      if (concepts.length !== conceptIds.length) {
        return res.status(404).json({ error: "One or more concepts not found" });
      }
      
      const idea = GateEngine.andGate(concepts as any);
      res.json({ success: true, idea });
    } catch (error) {
      res.status(500).json({ error: "Idea generation failed" });
    }
  }
  
  // Generate ideas using OR gate
  static async combineOR(req: Request, res: Response) {
    try {
      const { conceptIds } = req.body;
      if (!conceptIds || conceptIds.length < 2) {
        return res.status(400).json({ error: "Need at least 2 concepts for OR gate" });
      }
      
      const concepts = conceptIds.map((id: string) => ConceptService.getConceptById(id)).filter(c => c);
      const idea = GateEngine.orGate(concepts as any);
      res.json({ success: true, idea });
    } catch (error) {
      res.status(500).json({ error: "Idea generation failed" });
    }
  }
  
  // Generate ideas using NOT gate (A but NOT B)
  static async combineNOT(req: Request, res: Response) {
    try {
      const { includeConceptId, excludeConceptId } = req.body;
      const includeConcept = ConceptService.getConceptById(includeConceptId);
      const excludeConcept = ConceptService.getConceptById(excludeConceptId);
      
      if (!includeConcept || !excludeConcept) {
        return res.status(404).json({ error: "Concept(s) not found" });
      }
      
      const idea = GateEngine.notGate(includeConcept, excludeConcept);
      res.json({ success: true, idea });
    } catch (error) {
      res.status(500).json({ error: "Idea generation failed" });
    }
  }
  
  // Discovery endpoint: auto-combine random concepts from different domains
  static async randomDiscovery(req: Request, res: Response) {
    try {
      const allConcepts = ConceptService.getAllConcepts();
      const shuffled = [...allConcepts].sort(() => 0.5 - Math.random());
      const randomPair = shuffled.slice(0, 2);
      
      const idea = GateEngine.andGate(randomPair);
      res.json({ success: true, idea });
    } catch (error) {
      res.status(500).json({ error: "Random discovery failed" });
    }
  }
  
  // Generate batch of ideas (for entrepreneurs/researchers)
  static async batchGenerate(req: Request, res: Response) {
    try {
      const { conceptIds, gates = ["AND", "OR", "NOT"] } = req.body;
      const concepts = conceptIds.map((id: string) => ConceptService.getConceptById(id)).filter(c => c);
      
      const results: ScoredIdea[] = [];
      
      if (gates.includes("AND") && concepts.length >= 2) {
        results.push(GateEngine.andGate(concepts));
      }
      if (gates.includes("OR") && concepts.length >= 2) {
        results.push(GateEngine.orGate(concepts));
      }
      if (gates.includes("NOT") && concepts.length >= 2) {
        results.push(GateEngine.notGate(concepts[0], concepts[1]));
      }
      
      // Sort by novelty score descending
      results.sort((a, b) => b.noveltyScore - a.noveltyScore);
      
      res.json({ success: true, ideas: results });
    } catch (error) {
      res.status(500).json({ error: "Batch generation failed" });
    }
  }
}

7. API Routes (src/routes/apiRoutes.ts)
import { Router } from "express";
import { IdeaController } from "../controllers/ideaController";
import { ConceptService } from "../services/conceptService";

const router = Router();

// Idea generation endpoints
router.post("/combine/and", IdeaController.combineAND);
router.post("/combine/or", IdeaController.combineOR);
router.post("/combine/not", IdeaController.combineNOT);
router.get("/discover/random", IdeaController.randomDiscovery);
router.post("/generate/batch", IdeaController.batchGenerate);

// Concept management endpoints
router.get("/concepts", (req, res) => {
  res.json(ConceptService.getAllConcepts());
});

router.get("/concepts/search", (req, res) => {
  const query = req.query.q as string;
  if (!query) return res.status(400).json({ error: "Query param 'q' required" });
  res.json(ConceptService.searchConcepts(query));
});

router.post("/concepts", (req, res) => {
  const newConcept = ConceptService.addConcept(req.body);
  res.status(201).json(newConcept);
});

export default router;

8. Main App (src/app.ts)
import express from "express";
import cors from "cors";
import apiRoutes from "./routes/apiRoutes";

const app = express();
const PORT = process.env.PORT || 4000;

app.use(cors());
app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    name: "Idpedia API",
    version: "1.0.0",
    description: "Dynamic idea generator using AND/OR/NOT gates on concept spaces",
    endpoints: [
      "POST /combine/and",
      "POST /combine/or", 
      "POST /combine/not",
      "GET /discover/random",
      "POST /generate/batch",
      "GET /concepts",
      "POST /concepts"
    ]
  });
});

app.use("/api", apiRoutes);

app.listen(PORT, () => {
  console.log(`Idpedia backend running at http://localhost:${PORT}`);
});

9. Example API Usage
AND Gate Request
POST /api/combine/and
{
  "conceptIds": ["c1", "c2"]
}
Response:
json
Copy
Download
{
  "success": true,
  "idea": {
    "id": "and_1732345678_abc123",
    "name": "biodegradable materials + wireless charging hybrid",
    "description": "System integrating biodegradable materials and wireless charging. Emergent property: materials-driven electronics solution.",
    "noveltyScore": 82,
    "combinedConcepts": ["biodegradable materials", "wireless charging"],
    "gateUsed": "AND"
  }
}
NOT Gate Request
POST /api/combine/not
{
  "includeConceptId": "c4",
  "excludeConceptId": "c6"
}
→ “Self-healing polymers without blockchain” → novelty score 91.

